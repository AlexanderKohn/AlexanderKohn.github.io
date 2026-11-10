---
layout: post
title: "Testing the frequency lottery in grokking"
short_title: "The frequency lottery in grokking"
description: "Does the random draw at initialization decide which frequency each neuron learns when a small network groks modular addition? About 400 small runs on a desktop CPU say: on full data mostly yes, on a grokking split mostly no, and an initialization rigged by hand makes the network grok 5–9× faster."
image: /assets/img/grokking/interventions.png
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

I tested this with about 400 small training runs on my desktop CPU. Here's the short version of what I found:

- On full data, the initial weights largely decide each neuron's frequency: rerun from the same start with a
  different warmup, or even a different optimizer, and 81–82% of neurons land in the same place. But the paper's
  score only predicts about half of them.
- On a grokking split, they don't. Train the same initial weights on a different half of the pairs and only 16% of
  neurons end on the same frequency.
- The draw doesn't explain why width matters. Seeds with a "luckier" initialization don't grok any faster, and the
  width effect isn't just a larger effective learning rate either.
- But if you *rig* the initialization, giving every neuron a dominant, well-aligned frequency, the network groks
  5–9× faster, regardless of which frequencies you hand out.

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

## The initial weights decide more than the score can see

<figure>
  <a href="{{ '/assets/img/grokking/predictor_accuracy.png' | relative_url }}"><picture>
    <source media="(max-width: 34rem)" srcset="{{ '/assets/img/grokking/predictor_accuracy_narrow.png' | relative_url }}" width="846" height="1105">
    <img src="{{ '/assets/img/grokking/predictor_accuracy.png' | relative_url }}" alt="Bar chart. Trained on all pairs, the paper's score predicts about 44% of neurons at every initialization scale and the magnitude-only score about 25%. On the grokking split both fall to 12–22%. Chance is 4%." width="1726" height="585">
  </picture></a>
  <figcaption>How often a prediction made from the initial weights matches each neuron's final frequency (width 512, three seeds pooled; bars show 95% bootstrap intervals).</figcaption>
</figure>

Scored on the initial weights, the paper's predictor does clearly better than chance, but not by as much as I
expected. It's right for 44% of neurons when training on all the data, and only 14–22% on the grokking split. Odder
still, shrinking the initialization (which the theory assumes is small) made it *worse*.

The neurons commit very early, though. Checkpointing every 10 steps, the typical neuron has settled on its final
frequency within 10–20 steps on full data, and by around step 90 on the grokking split, well before the training
set is even fit:

<figure class="narrow">
  <a href="{{ '/assets/img/grokking/lock_in.png' | relative_url }}"><picture>
    <source media="(max-width: 34rem)" srcset="{{ '/assets/img/grokking/lock_in_narrow.png' | relative_url }}" width="845" height="622">
    <img src="{{ '/assets/img/grokking/lock_in.png' | relative_url }}" alt="Cumulative curves for three initialization scales, nearly identical. About 20% of neurons have settled by step 10, 65% by step 200 when the training set is fit, and 95% by step 5,650 when the network generalizes." width="1285" height="622">
  </picture></a>
  <figcaption>When each neuron settles on its final frequency for good, on the grokking split (width 512, three seeds per curve).</figcaption>
</figure>

So if the winner is decided within the first hundred or so steps, but the score only gets it right about half the
time, is the starting point really what decides it? A first look is to rerun the same score on the checkpoints
*after* step 0:

<figure>
  <a href="{{ '/assets/img/grokking/predictor_over_time.png' | relative_url }}"><picture>
    <source media="(max-width: 34rem)" srcset="{{ '/assets/img/grokking/predictor_over_time_narrow.png' | relative_url }}" width="842" height="1105">
    <img src="{{ '/assets/img/grokking/predictor_over_time.png' | relative_url }}" alt="Line chart. Trained on all pairs, accuracy rises from 44% at step 0 to 57–85% at step 10, depending on initialization scale, and to 95–99% by step 200. On the grokking split it rises from 14–22% to about 70% by step 200." width="1719" height="585">
  </picture></a>
  <figcaption>The same prediction, made from the weights at a later step instead of from the initial weights.</figcaption>
</figure>

On full data, ten Adam steps lift the prediction from 44% to 85% at small initialization. My first reading was that
Adam's early updates, which are roughly the same size whatever the weights are, overwrite the initial draw. That
turned out to be wrong. A score that gets better as training moves the weights toward their answer doesn't show
that the answer changed. The direct test is to keep the initial weights and the data fixed, change one thing about
the training, and see how many neurons still end on the same frequency:

<figure class="narrow">
  <a href="{{ '/assets/img/grokking/same_init.png' | relative_url }}"><picture>
    <source media="(max-width: 34rem)" srcset="{{ '/assets/img/grokking/same_init_narrow.png' | relative_url }}" width="875" height="647">
    <img src="{{ '/assets/img/grokking/same_init.png' | relative_url }}" alt="Horizontal bar chart. Trained on all pairs, 81% of neurons end on the same frequency with a warmup of 1 step versus 1,000 steps, and 82% with Adam versus plain gradient descent. Trained on half the pairs, the figures are 70% and 50%, and only 16% when the network is trained on a different half. Chance is about 4%." width="1284" height="547">
  </picture></a>
  <figcaption>Same initial weights, one thing changed. Share of neurons active in both runs that end on the same frequency (width 512, three seeds pooled).</figcaption>
</figure>

On full data, a 1,000-step warmup or a switch to plain gradient descent leaves about 80% of the outcome unchanged,
so the initial weights do largely decide it. What falls short is the paper's score, which captures only about half
of that. Under plain gradient descent, the optimizer the theory was derived for, the score does a little better
(52–53% against Adam's 44%), but not much. The early climb in the figure above tracks how far the weights have
moved: with a 1,000-step warmup, the jump that took ten steps takes about a hundred.

The grokking split is a different story. Train the same initial weights on a different half of the pairs, and only
16% of neurons end on the same frequency, against 4% by chance. The optimizer's path matters too: plain gradient
descent and Adam agree on only half of the neurons. About 30% of neurons switch frequency between memorizing and
generalizing, where on full data almost none do. Fitting half the data pushes neurons around in a way that fitting
all of it doesn't. This holds at widths 128, 256 and 512, and a smaller initialization makes the outcome even less
tied to the starting point (9% agreement at 0.1× against 16% at 1×), which is why smaller inits looked less
predictable.

One thing I first thought was surprising turned out not to be. The network uses all 23 frequencies, about evenly,
even long after grokking, whereas transformers are usually reported to settle on a handful. But that's exactly what
the analytic solution for this kind of MLP looks like: [Gromov (2023)](https://arxiv.org/abs/2301.02679) assigns one
frequency per neuron across the full set, and notes that extra width just adds redundant copies. My guess is that the
transformer's sparsity comes from its shared embedding layer, which can only fit a few frequencies, but I haven't
tested that yet.

## Width matters a lot, but not through the lottery

<figure>
  <a href="{{ '/assets/img/grokking/width_sweep.png' | relative_url }}"><picture>
    <source media="(max-width: 34rem)" srcset="{{ '/assets/img/grokking/width_sweep_narrow.png' | relative_url }}" width="848" height="1621">
    <img src="{{ '/assets/img/grokking/width_sweep.png' | relative_url }}" alt="Three dot plots against hidden width, 4 to 512. Test accuracy stays below 50% up to width 64 and jumps to 97% or more from width 128. Steps to 95% test accuracy fall from about 28,000 at width 128 to about 4,600 at width 512. The initial weights cover 21–23 of the 23 frequencies at width 64 and all 23 in nearly every seed from width 128." width="1728" height="565">
  </picture></a>
  <figcaption>Eight widths, eight seeds each; every dot is one run. Right: how many distinct frequencies the initial weights cover, by the paper's score, against what a random draw would give.</figcaption>
</figure>

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

There's an obvious confound. With Adam at a fixed learning rate, every weight moves by about the same amount per
step, so a wider network's output simply changes faster, and the width effect could be an effective learning rate
rather than anything about frequencies. A first pass seemed to confirm that: scale width 128's learning rate up by 4×
and it groks about as fast as width 512. But raising the learning rate in AdamW also raises the weight decay per step,
which is learning rate × weight decay. Holding that fixed separates the two:

<figure>
  <a href="{{ '/assets/img/grokking/lr_width.png' | relative_url }}"><picture>
    <source media="(max-width: 34rem)" srcset="{{ '/assets/img/grokking/lr_width_narrow.png' | relative_url }}" width="846" height="1105">
    <img src="{{ '/assets/img/grokking/lr_width.png' | relative_url }}" alt="Two line charts of steps to 95% test accuracy against learning rate for widths 128, 256 and 512. With weight decay fixed at 1, every width speeds up steeply with learning rate. With the decay per step fixed, the learning rate helps much less, and width 512 stays 4–6 times faster than width 128 at every learning rate." width="1726" height="585">
  </picture></a>
  <figcaption>Five seeds per point; lines join the medians. Settings where some runs never reached 95% are left out.</figcaption>
</figure>

With the decay per step fixed, an 8× higher learning rate speeds width 128 up only 2.5×, and it never catches
width 512. Across all 136 runs that generalized, steps to 95% scale roughly as width<sup>−1.0</sup> ×
learning rate<sup>−0.4</sup> × (decay per step)<sup>−0.8</sup>. So width isn't standing in for the learning rate.
From width 128 to 512 the time falls roughly in proportion to width (unevenly: most of the gain comes between 128
and 256), and the gain flattens by 1,024. And how fast weight
decay acts matters nearly as much as width. The threshold survives too: width 64 never reaches 95% even at a 16×
learning rate, though its plateau rises from 30–48% to 77–89%.

## Rigging the lottery

To get at causation, I took width 128, the narrowest width that generalizes, edited the initial weights, and
retrained on the same data:

- **Knockout:** remove the five frequencies the network ended up relying on most, from every neuron.
- **Monoculture:** force every neuron onto a single frequency.
- **Rigged:** give each neuron one strong frequency (twice its largest starting component) with its phases
  perfectly aligned. The frequencies are handed out either evenly or at random.

<figure>
  <a href="{{ '/assets/img/grokking/interventions.png' | relative_url }}"><picture>
    <source media="(max-width: 34rem)" srcset="{{ '/assets/img/grokking/interventions_narrow.png' | relative_url }}" width="848" height="2001">
    <img src="{{ '/assets/img/grokking/interventions.png' | relative_url }}" alt="Three panels of test accuracy against training step. Removing frequencies leaves the curves on top of the unedited runs. Putting all neurons on one frequency shifts them later. The rigged initializations rise from step 100 and reach 95% several times sooner than the unedited runs." width="1683" height="659">
  </picture></a>
  <figcaption>Test accuracy over training for each edit at width 128, three seeds per edit. The dashed line is 95%.</figcaption>
</figure>

| edit to the initial weights | steps to 95% test accuracy (3 seeds) |
|---|---|
| none | 31.9k / 24.0k / 27.8k |
| remove the top 5 frequencies | 26.2k / 23.1k / 29.5k |
| everything on one frequency | 4 of 9 runs never reach 95% |
| rigged, frequencies spread evenly | **11.2k / 3.0k / 3.5k** |
| rigged, frequencies assigned at random | **4.3k / 4.9k / 6.2k** |
| rigged at random, 4× instead of 2× | **3.1k / 2.4k / 4.8k** |
| rigged at random, 1× (ties the neuron's largest component) | 25.2k / 29.4k / 21.1k |
| 2× boost, but output phase random (not aligned) | 13.5k / 20.1k / 15.2k |
| phases aligned, no boost | 27.3k / 27.3k / 34.9k |

The knockout did nothing; the network just regrew the missing frequencies. Monoculture hurt: one frequency is
enough to solve the task in principle, but the network learned more slowly and spread back out to 21–23 frequencies
anyway.

Rigging helped a lot. Test accuracy starts rising almost immediately, and the long plateau mostly disappears. The
even and random assignments are equally fast, so what matters isn't which frequencies each neuron gets, but that
every neuron starts out as a clean, aligned feature.

The last four rows pull that apart. The boost does nothing until the assigned frequency is clearly the largest one in
the neuron: at 1× it only ties, and the runs are as slow as unedited ones. Alignment alone does nothing either, since
the paper's winner is usually close to aligned already. A 2× boost with a random output phase helps a little (about
1.8×), and the same boost aligned helps about 4×. So it takes both: one frequency has to dominate each neuron, and its
phases have to line up.

I hoped one number would tie this to width: the strength of the coherent Fourier signal summed over neurons at
initialization. It doesn't. Within width 128 it predicts the rigged runs moderately well (R² ≈ 0.5), but across
widths a much smaller increase in it goes with a much larger speed-up, so width must be working through something
else.

This fits a few things people have seen before:

- [Varma et al. (2023)](https://arxiv.org/abs/2309.02390) describe grokking as a generalizing circuit that's more
  efficient but slower to learn than memorization. Rigging gives that circuit a head start.
- [GrokTransfer (Xu et al., 2025)](https://arxiv.org/abs/2504.13292) shows that initializing from a smaller model's
  learned embedding can nearly eliminate the grokking delay. What this adds is *which* property of the
  initialization does the work: a dominant frequency per neuron, phase-aligned. Without the alignment it gets a
  fraction of the speed-up, and alignment without the dominant frequency gets none.
- Sparse parity behaves differently. There, [Edelman et al. (2023)](https://arxiv.org/abs/2309.03800) find that
  width works like a parallel search for rare "winning ticket" neurons. Modular addition is a dense task that wants
  the whole spectrum, so there's no rare winner to find, which may be why luck in the draw doesn't matter.

## Where this leaves me

The lottery mechanism is real: frequencies with stronger, better-aligned starting components do tend to win, and
you can exploit that to make a network grok several times faster. On full data the draw at initialization largely
decides each neuron's frequency, whichever optimizer you use, but the paper's score captures only about half of
that. On a grokking split the draw matters much less: which pairs are held out, and the path through memorization, reassign
most neurons. Either way the network routes around attempts to take frequencies away, and luck in the draw doesn't
change how fast it groks.

Width is the bigger lever, and it isn't an effective learning rate in disguise. What it does instead is still open:
the coherent starting signal I measured doesn't account for it.

## What's next

Two of the planned follow-ups are still open:

- a sweep over *p*, to see whether the width threshold scales with the number of frequencies;
- an embedding bottleneck, to see whether it produces transformer-style sparsity.

Plain gradient descent mostly didn't generalize on the split within 50,000 steps (one run of six reached 95%), so a
fair test of the theory on its home ground would need much longer runs, or the paper's exact architecture.

Everything here ran on a desktop CPU in PyTorch, at p = 47, with 3–8 seeds per condition (5 for the learning-rate
sweep), so treat the numbers as small-sample.

*Updated November 9, 2026.* The first version of this post said that on full data Adam's first few steps, rather
than the initial weights, decide each neuron's frequency, and left the width effect's learning-rate confound open.
Follow-up runs showed the first claim was wrong and closed the second.

The code, the plan for the follow-ups and every number above are in
[the repo](https://github.com/AlexanderKohn/grokking_experiments). The detailed write-up is
[`RESULTS.md`](https://github.com/AlexanderKohn/grokking_experiments/blob/master/experiments/01_frequency_lottery/RESULTS.md).
