---
layout: post
title: "The frequency lottery in grokking: a real mechanism, an irrelevant draw"
short_title: "The frequency lottery in grokking"
description: "Does random initialization decide which Fourier feature each neuron learns when a small MLP groks modular addition, and is that why width matters? About 130 CPU training runs say mostly no, with one twist."
---

Small networks trained on modular addition famously *grok*. They memorize the training set almost immediately, sit
at chance on held-out pairs for thousands of steps, and then abruptly generalize. Mechanistic work has shown what
they learn when they do: each hidden neuron specializes in a single Fourier frequency, and the network combines
them into a "clock" algorithm for adding mod *p*.

A recent paper (arXiv 2602.16849) proposes a **lottery-ticket mechanism** for how neurons pick their frequency.
Under small random initialization, the frequencies compete inside each neuron. The winner is the one that starts with
the largest magnitude and the best-aligned phases. If that's right, the random draw at initialization should
largely determine the final algorithm. It would also suggest a tidy story for why **wider networks grok faster**:
more neurons means more lottery tickets, so it's more likely that every frequency the algorithm needs gets covered.

I tested that story on my own desktop CPU. The short version:

- **The lottery is real but weak.** Initialization predicts a neuron's final frequency at 3–10× chance, but most
  neurons don't end up on their predicted winner.
- **Neurons commit almost immediately.** Most settle on their final frequency within the first ~100 steps, long
  before the network generalizes. About 30% switch frequency between memorizing and grokking.
- **Natural variation in the draw doesn't affect outcomes.** At a fixed width, seeds with better initial
  "coverage" don't grok faster.
- **Coverage doesn't explain why width matters.** Frequency coverage at initialization is already complete at the
  narrowest width that groks, yet grokking keeps getting 6× faster as width grows.
- **The twist:** the lottery's *mechanism* is powerful when you push it. Initializations with strong, phase-aligned
  Fourier features grok **5–9× faster**, however those features are spread across frequencies.

## Setup

- **Task and model:** a 1-hidden-layer ReLU MLP takes one-hot encodings of *a* and *b* and predicts *a + b* mod 47.
  It trains on 50% of the 2,209 possible pairs, full-batch, with AdamW (lr 10⁻³, weight decay 1.0). This is the
  standard grokking setup; at width 512 it memorizes in about 200 steps and generalizes at about 5,500.
- **Choice of p:** I used p = 47 rather than the usual 97 or 113 so that a sweep over 8 widths × 8 seeds fits on a
  CPU. The whole project, about 130 training runs, ran on 6 cores in a few hours.
- **Input format:** the one-hot input matters. The same MLP given *a* and *b* as raw numbers never even memorizes,
  because nothing in two scalars tells it about the wrap-around at *p*.

![transformer vs. MLP baselines]({{ '/assets/img/grokking/compare_models_add_p97.png' | relative_url }})
*Baselines at p = 97, from before the main experiment. Top: the classic 1-layer transformer. Middle: the one-hot MLP
used here. Bottom: the raw-integer MLP, which never fits.*

**Measuring which frequency a neuron uses.** Each hidden neuron *m* has a row of input weights for *a*, another for
*b*, and a column of output weights. Taking the discrete Fourier transform of each gives, for every frequency *k*,
a magnitude and a phase:

- **Dominant frequency k\*:** the frequency with the most combined power in neuron *m*.
- **Phase alignment D = φₐ + φ_b − ψ:** this combination of the input phases (φₐ, φ_b) and output phase (ψ) has to be
  near 0 for the neuron to vote for the correct answer *a + b*.

On every trained network, D concentrates completely at 0 across neurons. The theory's sign convention, adapted to
separate input weights for *a* and *b*, checks out.

**The init-time predictor.** Following the paper, I score each frequency by magnitude × phase alignment at
initialization and predict that the top score wins.

## 1. The lottery predicts, but weakly

![predictor accuracy]({{ '/assets/img/grokking/phase1b_predictor_accuracy.png' | relative_url }})
*Fraction of active neurons whose final frequency matches the init-time prediction. Width 512, 3 seeds per
condition. The dashed line is chance (1/23).*

- **Full training data (no grokking):** the magnitude × alignment predictor gets **44%** of neurons right, against
  4.3% chance. Raw magnitude alone does much worse (24%).
- **Grokking split:** accuracy drops to **14–22%**.
- **Init scale:** the theory assumes small initialization, but shrinking the init made prediction *worse* (14% at
  scale 0.1 vs. 22% at 1.0). My best guess is the optimizer. The analysis is for plain gradient descent, while Adam's
  early updates have a fixed size regardless of weight scale, so with a small init they swamp it.

![lock-in]({{ '/assets/img/grokking/phase1b_lockin.png' | relative_url }})
*When does each neuron settle on its final frequency for good? Dotted line: training set fit. Dashed line: test set
generalized.*

Neurons commit very early. With checkpoints every 10 steps, the median neuron has locked in its final frequency by
**step ~90**, and about 65% have locked in before the training set is even fit. Yet the init-time predictor tops out
well below that. So the first few dozen optimizer steps are doing a lot of the choosing, beyond what the initial
weights encode.

Grokking also **reshuffles** neurons. About 30% change frequency between memorizing and generalizing, against
≤ 6% when training on all the data.

**A surprise: no "key frequencies."** Transformers that grok modular addition are usually reported to end up using
just 3–6 frequencies. This MLP uses **all 23**, nearly uniformly, even 30,000 steps after grokking. Weight decay
sharpens individual neurons onto one frequency each, but never prunes the population down to a few.

## 2. Width matters a lot, and coverage doesn't explain it

![width sweep]({{ '/assets/img/grokking/phase1c_width_sweep.png' | relative_url }})
*8 widths × 8 seeds. Left: fraction of seeds that generalize within 50k steps. Middle: steps to 95% test accuracy.
Right: how many frequencies the initial weights "cover", against the coupon-collector expectation.*

There's a sharp threshold:

| width | test accuracy | steps to 95% test accuracy |
|---|---|---|
| 4–64 | 1–48% (plateaus) | never, within 50k steps |
| 128 | 97–99% | ~27,600 |
| 256 | 100% | ~7,900 |
| 512 | 100% | ~4,600 |

**The coverage story fails.** If generalizing required the lottery winners to cover the spectrum, the threshold
would sit where coverage completes.

- At width 64, the initial winners already cover 22 of 23 frequencies, yet the network stalls at 39% test accuracy.
- From width 128 up, coverage is complete in every seed, yet grokking still gets 6× faster from 128 to 512.

**Within-width test.** I also asked the more direct question: at a fixed width, do seeds whose initial draw covers
more frequencies, or stacks more neurons per frequency, grok faster? Across six init-time statistics and three
outcome measures, none predicted anything. The combined permutation tests gave every p ≥ 0.25. With 8 seeds per
width, this only rules out fairly strong effects, but the spread between seeds is also small (about ±10%).

One thing does correlate, though only after training. Below the threshold, networks that end up concentrating
**more neurons on fewer frequencies** reach higher test accuracy (p = 0.003). Redundancy seems to be the scarce
resource, not coverage.

## 3. Interventions: editing the lottery directly

To test causation, I took width 128, the narrowest width that generalizes, and edited the initial weights before
training:

- **Knockout:** remove the run's 5 most-used frequencies from every neuron, keeping each neuron's weight norm.
- **Monoculture:** project every neuron onto a single frequency.
- **Rigged:** give each neuron one frequency at twice its largest initial magnitude, with phases perfectly aligned.
  The frequencies are assigned either round-robin (perfectly even coverage) or at random.

Each edited network trained on the same data split as its unedited baseline, which reproduced the original run
exactly.

![interventions]({{ '/assets/img/grokking/phase1d_curves.png' | relative_url }})
*Test accuracy over training for each edit, 3 seeds (rows). The dotted line is 95%.*

| edit to the initial weights | steps to 95% test accuracy (3 seeds) |
|---|---|
| none | 31.9k / 24.0k / 27.8k |
| remove top-5 frequencies | 26.2k / 23.1k / 29.5k |
| single frequency | 4 of 9 runs never reach 95% |
| rigged, round-robin | **11.2k / 3.0k / 3.5k** |
| rigged, random assignment | **4.3k / 4.9k / 6.2k** |

- **Knockout does nothing.** The removed frequencies simply **grow back** during training, to 12–14% of the final
  network. Whichever frequencies a network ends up using, they aren't fixed by its initialization.
- **Monoculture hurts.** One frequency is enough to solve the task in principle, but forcing it slows learning, and
  the network spreads back out to 21–23 frequencies anyway.
- **Rigging helps enormously, but not through coverage.** With strong, aligned features at the start, test accuracy
  starts climbing almost immediately and the long plateau mostly disappears. Random assignment is just as fast as
  perfectly even coverage.

## What I take away

The lottery-ticket *mechanism* is real: frequencies with stronger, better-aligned starting components do tend to
win, and you can exploit that to make a network grok 5–9× faster. But the natural draw from a standard random
initialization is too weak to matter:

- it predicts only a minority of final assignments;
- seeds differ too little for it to change outcomes;
- the network routes around attempts to take frequencies away.

What seems to set grokking speed is **how strong the population's Fourier signal is to begin with**. Which particular
frequencies won, or how many were covered, matters little. On that reading, width helps by adding neurons per
frequency, which strengthens that starting signal. It doesn't help by covering more of the spectrum.

## Caveats and next steps

- **Sample sizes are small:** 3 seeds per intervention and 8 per width. The null result at a fixed width rules out
  only strong effects.
- **Everything is at p = 47 with AdamW.** The obvious next checks are a plain gradient-descent arm, since the theory
  is for gradient flow and Adam may be what blurs the lottery, and a confirmation at p = 97.
- **The rigged initialization changes several things at once:** it makes each neuron single-frequency, aligns its
  phases, and raises the overall Fourier signal. A sweep over the boost strength, plus alignment-only and
  concentration-only variants, would show which of these does the work.
- **The checkpoint grid had to change.** The originally planned schedule of 250-step checkpoints was far too coarse,
  because most of the frequency competition is over by step 250.

*All runs were on a desktop CPU (6 cores) in PyTorch.
**Code, specs and the full results write-up:**
[github.com/AlexanderKohn/grokking_experiments](https://github.com/AlexanderKohn/grokking_experiments). See
[`experiments/01_frequency_lottery/RESULTS.md`](https://github.com/AlexanderKohn/grokking_experiments/blob/master/experiments/01_frequency_lottery/RESULTS.md)
for every number above and the commands to reproduce them.*
