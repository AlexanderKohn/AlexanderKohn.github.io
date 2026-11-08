---
layout: default
---

## About

<div class="todo" markdown="1">
**TODO (Alex):** two or three sentences about who you are: your background, what you're working on now, and what
kind of role or program you're looking for. The 2018 repos below suggest a computational chemistry background; say
so here if that's right.
</div>

Lately I've been running small experiments, with the analysis plan written down before the runs, on two questions:

- **How networks learn:** what decides which features a model learns, and when it generalizes.
- **What models prefer:** whether language models' stated choices reflect real preferences.

I try to design each experiment so that it can come out against my hypothesis, and to write up what happened
either way.

## Writing

<ul class="post-list">
{% for post in site.posts %}
  <li>
    <a href="{{ post.url | relative_url }}"><strong>{{ post.title }}</strong></a>
    <span class="date">&middot; {{ post.date | date: "%B %Y" }}</span><br>
    {{ post.description }}
  </li>
{% endfor %}
</ul>

## Projects

**[Grokking experiments](https://github.com/AlexanderKohn/grokking_experiments)** (2026)
: A CPU-friendly testbed for grokking on modular arithmetic: transformer and MLP models, per-neuron Fourier
  analysis, and interventions on the initial weights. The first study, a test of the "frequency lottery" account of
  grokking over about 130 runs, is written up in the post above.

**[Preferential gaps vs. indifference in LLMs](https://github.com/AlexanderKohn/utility_engineering_sweetener_experiments)** (2026)
: When a language model splits 50/50 between two options, is it indifferent, or is there a gap that a small
  "sweetener" can't close? Uses exact next-token log-odds, an analysis plan fixed before collecting any model data,
  and positive and negative controls. Tested on Qwen2.5-7B, Qwen3-8B and Granite 4.2 8B; the second run found no
  general gaps, and a positive control failed, which the write-up reports as a limit of the setup.

**Earlier work** (2018)
: Python tools for computational chemistry: batch-submitting
  [Q-Chem jobs](https://github.com/AlexanderKohn/massQM),
  [QM/MM scripts](https://github.com/AlexanderKohn/QMMM-IPEA),
  and an [inverse participation ratio calculator](https://github.com/AlexanderKohn/IPRcalc).
  Also a [Monte Carlo sorter](https://github.com/AlexanderKohn/montecarloSort) that splits two-feature data into two
  linear fits, and the start of a [Magic: The Gathering simulator](https://github.com/AlexanderKohn/mtgsim) for
  testing deck-construction ideas.

## Contact

- GitHub: [AlexanderKohn](https://github.com/AlexanderKohn)

<div class="todo" markdown="1">
**TODO (Alex):** add whatever you want public: an email address, LinkedIn, Google Scholar, and a link to a CV PDF
(put it in `assets/` and link it here).
</div>
