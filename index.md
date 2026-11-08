---
layout: default
---

## About

I'm a data scientist with a Ph.D. in physical chemistry from MIT and eight years of production machine learning in
security and trust & safety. I worked on malware detection and behavioral authentication at BlackBerry Cylance, and
I now build models at Blizzard Entertainment that detect toxic chat, offensive player names and cheating hardware.
Since 2023 that work has included LLM labeling, fine-tuning and retrieval-based assistants.

**I'm interested in technical AI safety.** What my work has in common is careful evaluation, clean labels,
and models that hold up after deployment. Outside work, I've been running small independent experiments on two
questions:

- **How networks learn:** what decides which features a model learns, and when it generalizes.
- **What models prefer:** whether language models' stated choices reflect real preferences.

I write the analysis plan down before running anything, design experiments that can come out against my hypothesis,
and report what happened either way.

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

## Independent research

**[Grokking experiments](https://github.com/AlexanderKohn/grokking_experiments)** (2026)
: A CPU-friendly testbed for grokking on modular arithmetic: transformer and MLP models, per-neuron Fourier
  analysis, and interventions on the initial weights. The first study, a test of the "frequency lottery" account of
  grokking over about 130 runs, is written up in the post above.

**[Preferential gaps vs. indifference in LLMs](https://github.com/AlexanderKohn/utility_engineering_sweetener_experiments)** (2026)
: When a language model splits 50/50 between two options, is it indifferent, or is there a gap that a small
  "sweetener" can't close? Uses exact next-token log-odds, an analysis plan fixed before collecting any model data,
  and positive and negative controls. Tested on Qwen2.5-7B, Qwen3-8B and Granite 4.2 8B; the second run found no
  general gaps, and a positive control failed, which the write-up reports as a limit of the setup.

## Experience

### Blizzard Entertainment, Player Interactions and Trust (2021–present)

Senior Data Scientist (2022–present); Data Scientist (2021–2022).

- **Harmful-content classifiers:** I own the production text classifiers that screen player names and in-game chat
  in games with millions of daily players, from data collection through deployment and retraining. The player-name
  model, now in its fourth major version, reaches 95% recall at 99% precision. I've added language coverage and
  improved chat accuracy through distillation and label cleaning.
- **Policy-driven chat classifier:** a nine-category classifier covering toxicity, hate speech, prosocial behavior,
  spam and phishing. I wrote a policy definition for each category, had an LLM label real chat against them, and
  fine-tuned Llama 3.1 8B with LoRA. It is now part of analysts' review workflows.
- **Cheat detection:** I improved the production detector for hardware cheat devices. I also built a Beta-Binomial
  model of how reliable each player report is, and combined reports into a per-player score for ranking suspects.
- **LLM assistants:** I built the company's first internal LLM chat agents, a retrieval-augmented game-lore
  assistant and a customer-support assistant, and took both from prototype to production.
- **Causal analysis:** I used double machine learning to estimate how exposure to cheating and toxicity affects
  player engagement.

### BlackBerry Cylance, Data Scientist, Research (2018–2021)

- **Persona:** one of four developers of an unsupervised system that continuously authenticates users from their
  behavior. I wrote its Bayesian models and evaluation pipeline, and I'm a co-inventor on the patent.
- **Malware and adversarial robustness:** I built LightGBM models that classify malicious PHP (97.1% accuracy) and researched defenses against adversarial
  string attacks on a production malware classifier.

### MIT, Van Voorhis Lab, Graduate Researcher (2013–2018)

Excited-state electronic structure models of non-radiative decay in organic electronics. I also used a graph neural
network to predict tuning parameters for molecules, and wrote Python pipelines for electronic structure
calculations. Some of the [tooling from that era](#earlier-code) is on GitHub.

## Publications and patent

- A. W. Kohn, Z. Lin, T. Van Voorhis. *Toward Prediction of Nonradiative Decay Pathways in Organic Compounds I: The
  Case of Naphthalene Quantum Yields.* J. Phys. Chem. C, 2019.
- Z. Lin, A. W. Kohn, T. Van Voorhis. *Toward Prediction of Nonradiative Decay Pathways in Organic Compounds II: Two
  Internal Conversion Channels in BODIPYs.* J. Phys. Chem. C, 2020.
- A. W. Kohn, D. McMahon, S. Wen, T. Van Voorhis. *The Impact of Carrier Delocalization and Interfacial Electric
  Field Fluctuations on Organic Photovoltaics.* J. Phys. Chem. C, 2017.
- M. T. Wojnowicz, D. H. Nguyen, A. W. Kohn. *Bayesian Continuous User Authentication.*
  [U.S. Patent 11,544,358 B2](https://patents.google.com/patent/US11544358B2/en), 2023.

Full list on [Google Scholar](https://scholar.google.com/citations?user=Y4YtkbcAAAAJ).

## Education

- **Ph.D., Physical Chemistry**, Massachusetts Institute of Technology, 2018
- **B.S., Chemistry**, *summa cum laude*, Harvey Mudd College, 2013

## Earlier code

Python tools from 2018:

- **Computational chemistry:** [batch Q-Chem jobs](https://github.com/AlexanderKohn/massQM),
  [QM/MM scripts](https://github.com/AlexanderKohn/QMMM-IPEA) and an
  [inverse participation ratio calculator](https://github.com/AlexanderKohn/IPRcalc).
- **Other:** a [Monte Carlo sorter](https://github.com/AlexanderKohn/montecarloSort) that splits two-feature data
  into two linear fits, and the start of a [Magic: The Gathering simulator](https://github.com/AlexanderKohn/mtgsim)
  for testing deck-construction ideas.

## Contact

- Email: [awolfekohn@gmail.com](mailto:awolfekohn@gmail.com)
- LinkedIn: [alexander-kohn](https://www.linkedin.com/in/alexander-kohn/)
- GitHub: [AlexanderKohn](https://github.com/AlexanderKohn)
- Google Scholar: [Alexander W. Kohn](https://scholar.google.com/citations?user=Y4YtkbcAAAAJ)
- Based in Irvine, CA
