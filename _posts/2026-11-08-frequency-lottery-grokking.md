---
layout: post
title: "The frequency lottery in grokking: a real mechanism, an irrelevant draw"
short_title: "The frequency lottery in grokking"
description: "Does the random draw at initialization decide which frequency each neuron learns when a small network groks modular addition? I ran about 130 small experiments on a desktop CPU to find out. Mostly it doesn't, with one twist."
---

If you train a small network to add numbers mod *p*, something odd happens. It memorizes the training set almost
immediately, then sits at chance on held-out pairs for thousands of steps, and then, quite suddenly, it gets them
right. This is *grokking*. We also know a fair amount about what the network has learned once it gets there: each
hidden neuron latches onto a single Fourier frequency, and together they implement a little clock-arithmetic
algorithm.

What I wanted to understand was how each neuron ends up with its frequency. A recent paper
([He et al., 2026](https://arxiv.org/abs/2602.16849)) argues it's a lottery. At initialization, every neuron has a
little bit of every frequency in its weights. Those frequencies compete, and the one that starts out biggest and
best-aligned wins.

That's a nice story, and it suggests an explanation for something else: wider networks grok faster. More neurons
means more lottery tickets, so you're more likely to end up with every frequency the algorithm needs.

I tested this on my desktop CPU over a few hours of training. Here's the short version of what I found:

- The lottery is real, but the random draw at initialization doesn't decide it. On full data, Adam's first few
  steps largely do. On a grokking split, it gets decided gradually during memorization.
- The draw doesn't explain why width matters. Seeds with a "luckier" initialization don't grok any faster.
- But if you *rig* the initialization, giving every neuron a strong, well-aligned frequency, the network groks 5–9×
  faster, regardless of which frequencies you hand out.

## The setup

The model is a one-hidden-layer MLP that gets *a* and *b* as one-hot vectors and predicts *a + b* mod 47. It trains
on half of the 2,209 possible pairs with full-batch AdamW and weight decay 1.0, a standard grokking setup. At width
512 it memorizes in about 200 steps and generalizes around step 5,500. I used p = 47 instead of the usual 97 or 113
so that sweeping 8 widths × 8 seeds would fit on a CPU.

To see which frequency a neuron uses, I take the Fourier transform of its input weights (one set for *a*, one for
*b*) and its output weights. Its dominant frequency is the one with the most power, and its phases tell you whether
it will actually vote for the right answer. For that, the input phases have to add up to the output phase, and in
every trained network they do, almost exactly.

To predict the winner from the initial weights, I followed the paper: score each frequency by its starting
magnitude times its phase alignment, and bet on the top score.

## The lottery is drawn early, and mostly not at initialization

![predictor accuracy]({{ '/assets/img/grokking/phase1b_predictor_accuracy.png' | relative_url }})
*How often the init-time prediction matches each neuron's final frequency (width 512, 3 seeds per bar). The dashed
line is chance, 1 in 23.*

Scored on the initial weights, the paper's predictor does clearly better than chance, but not by as much as I
expected. It's right for 44% of neurons when training on all the data, and only 14–22% on the grokking split. Odder
still, shrinking the initialization (which the theory assumes is small) made it *worse*.

The neurons commit very early, though. Checkpointing every 10 steps, the typical neuron has settled on its final
frequency within 10–20 steps on full data, and by around step 90 on the grokking split, well before the training
set is even fit:

![lock-in]({{ '/assets/img/grokking/phase1b_lockin.png' | relative_url }})
*When each neuron settles on its final frequency for good, on the grokking split. The dotted line is when the
training set is fit; the dashed line is when the network generalizes.*

So if the winner is decided within the first hundred or so steps, but the initial weights only predict it about
half the time, what decides it? I reran the same predictor on the checkpoints *after* step 0:

![predictor over time]({{ '/assets/img/grokking/phase1e_e1_predictor_over_time.png' | relative_url }})
*The same prediction made from the weights at step t instead of step 0.*

On full data, ten Adam steps are enough to lift the prediction from 44% to 85% at small initialization. Adam's
early updates are roughly the same size whatever the weights are, so when the initial weights are small, those
first steps overwrite them almost immediately. The "initial condition" that matters is the network after ten
steps, not the one you sampled. That also explains why smaller inits looked less predictable.

The grokking split behaves differently. Accuracy climbs slowly through memorization, reaching about 70% by step 200,
and about 30% of neurons switch frequency between memorizing and generalizing. On full data, almost none do. Fitting
half the data seems to push neurons around in a way that fitting all of it doesn't.

One thing I first thought was surprising turned out not to be. The network uses all 23 frequencies, about evenly,
even long after grokking, whereas transformers are usually reported to settle on a handful. But that's exactly what
the analytic solution for this kind of MLP looks like: [Gromov (2023)](https://arxiv.org/abs/2301.02679) assigns one
frequency per neuron across the full set, and notes that extra width just adds redundant copies. My guess is that the
transformer's sparsity comes from its shared embedding layer, which can only fit a few frequencies, but I haven't
tested that yet.

## Width matters a lot, but not through the lottery

![width sweep]({{ '/assets/img/grokking/phase1c_width_sweep.png' | relative_url }})
*8 widths × 8 seeds. Left: the fraction of seeds that generalize within 50k steps. Middle: steps to 95% test
accuracy. Right: how many distinct frequencies the initial weights "cover", against what random assignment
predicts.*

There's a sharp threshold. At width 64 and below, the network never generalizes; at width 64 it memorizes but stalls
at 30–48% test accuracy. At width 128 it gets to 97–99%, slowly (about 27,600 steps). At 256 it takes about 7,900 steps, and
at 512 about 4,600.

If the coverage story were right, the threshold would sit where the initial draw first covers every frequency. It
doesn't. At width 64 the initial weights already cover 22 of the 23 frequencies, and from 128 up every seed covers
all of them, yet grokking keeps getting 6× faster from 128 to 512.

The more direct test came out null too. At a fixed width, seeds whose initial draw covered more frequencies, or
stacked more neurons on each one, didn't grok any faster. With 8 seeds per width I can only rule out fairly strong
effects, but there also isn't much seed-to-seed variation to explain.

The one thing that did correlate was the *final* network. Below the threshold, networks that concentrated more
neurons on fewer frequencies generalized better. Redundancy looks like the scarce resource, rather than coverage.

I'd like to say width helps by adding neurons per frequency, but there's a confound I haven't ruled out. With Adam at
a fixed learning rate, every weight moves by about the same amount per step, so a wider network's output simply
changes faster. Some of the width effect could be an effective learning rate rather than anything about
frequencies. That's the next thing to test.

## Rigging the lottery

To get at causation, I took width 128, the narrowest width that generalizes, edited the initial weights, and
retrained on the same data:

- **Knockout:** remove the five frequencies the network ended up relying on most, from every neuron.
- **Monoculture:** force every neuron onto a single frequency.
- **Rigged:** give each neuron one strong frequency (twice its largest starting component) with its phases
  perfectly aligned. The frequencies are handed out either evenly or at random.

![interventions]({{ '/assets/img/grokking/phase1d_curves.png' | relative_url }})
*Test accuracy over training for each edit, one row per seed. The dotted line is 95%.*

| edit to the initial weights | steps to 95% test accuracy (3 seeds) |
|---|---|
| none | 31.9k / 24.0k / 27.8k |
| remove the top 5 frequencies | 26.2k / 23.1k / 29.5k |
| everything on one frequency | 4 of 9 runs never reach 95% |
| rigged, frequencies spread evenly | **11.2k / 3.0k / 3.5k** |
| rigged, frequencies assigned at random | **4.3k / 4.9k / 6.2k** |

The knockout did nothing; the network just regrew the missing frequencies. Monoculture hurt: one frequency is
enough to solve the task in principle, but the network learned more slowly and spread back out to 21–23 frequencies
anyway.

Rigging helped a lot. Test accuracy starts rising almost immediately, and the long plateau mostly disappears. The
even and random assignments are equally fast, so what matters isn't which frequencies each neuron gets, but that
every neuron starts out as a clean, aligned feature.

It helps to be precise about why. In a random initialization the phases are random, so the neurons don't add up to
any coherent signal: there's nothing there for width to amplify. Rigging creates that signal from the start.

This fits a few things people have seen before:

- **Circuit competition:** [Varma et al. (2023)](https://arxiv.org/abs/2309.02390) describe grokking as a
  generalizing circuit that's more efficient but slower to learn than memorization. Rigging just gives it a head
  start.
- **Structured initialization:** [GrokTransfer (Xu et al., 2025)](https://arxiv.org/abs/2504.13292) shows that
  initializing from a smaller model's learned embedding can nearly eliminate the grokking delay. What I'd like to
  add is *which* property of the initialization does the work. I can't separate concentration from alignment yet,
  but the next round of experiments is designed to.
- **A contrast with sparse parity:** there, [Edelman et al. (2023)](https://arxiv.org/abs/2309.03800) find that
  width works like a parallel search for rare "winning ticket" neurons. Modular addition seems different: it's a
  dense task that wants the whole spectrum, so there's no rare winner to find, which may be why luck in the draw
  doesn't matter.

## Where this leaves me

The lottery mechanism is real: frequencies with stronger, better-aligned starting components do tend to win, and
you can exploit that to make a network grok several times faster. But the draw you get from a normal random
initialization barely matters. With full data it's mostly redrawn by the optimizer in the first few steps; on a
grokking split, memorization reshuffles it; and either way the network routes around attempts to take frequencies
away.

Two of my explanations are still hypotheses. "Width helps by adding neurons per frequency" has to survive the
effective-learning-rate control. "Initial signal strength decides grokking time" needs a single measurable quantity
that every run (different widths, seeds and rigged variants) lines up on.

## What's next

The follow-ups are written up as a plan in the repo. Briefly, they are:

- plain gradient descent instead of Adam (the theory's own setting);
- a sweep over *p* to see whether the width threshold scales with the number of frequencies;
- an embedding bottleneck, to see whether it produces transformer-style sparsity;
- a width sweep with the learning rate rescaled per layer, to remove the effective-learning-rate confound;
- a dose-response on the rigged initialization that separates concentration from alignment.

Everything here ran on a desktop CPU in PyTorch, at p = 47, with 3–8 seeds per condition, so treat the numbers as
small-sample.
**The code, the plan for the follow-ups, and the full results with every number above are on GitHub:**
[github.com/AlexanderKohn/grokking_experiments](https://github.com/AlexanderKohn/grokking_experiments). The detailed
write-up is
[`RESULTS.md`](https://github.com/AlexanderKohn/grokking_experiments/blob/master/experiments/01_frequency_lottery/RESULTS.md).
