---
layout: default
---

## About

I'm a data scientist with a Ph.D. in physical chemistry from MIT and eight years of production ML in security and
trust & safety, at BlackBerry Cylance and now at Blizzard Entertainment. My work centers on careful evaluation, clean
labels, and models that hold up after deployment. Since 2023 that has included LLM labeling, fine-tuning and
retrieval-based assistants.

I'm interested in technical AI safety, and I've been running small independent experiments on how networks learn
and what language models prefer. [Résumé (PDF)]({{ '/assets/Kohn_Resume_2026.pdf' | relative_url }})

## Writing

<ul class="post-list">
{% for post in site.posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span class="date">&middot; {{ post.date | date: "%B %Y" }}</span><br>
    {{ post.description }}
  </li>
{% endfor %}
</ul>

## Independent research

- **[Grokking experiments](https://github.com/AlexanderKohn/grokking_experiments)** (2026): does the initial "frequency
  lottery" decide what a small network learns when it groks? Written up [above](#writing).
- **[Preferential gaps in LLMs](https://github.com/AlexanderKohn/utility_engineering_sweetener_experiments)** (2026):
  when a model splits 50/50 between two options, is it indifferent, or is there a gap a small sweetener can't close?

## Experience

**Blizzard Entertainment**, Senior Data Scientist, Player Interactions and Trust (2021–present)

- Own the production classifiers that screen player names and in-game chat for harmful content, in games with
  millions of daily players. The name model reaches 95% recall at 99% precision.
- Built a policy-driven chat classifier by having an LLM label chat against written category definitions, then
  fine-tuning Llama 3.1 8B with LoRA.
- Built the company's first internal LLM assistants and took them to production.

**BlackBerry Cylance**, Data Scientist, Research (2018–2021)

- Co-developed *Persona*, a Bayesian system that continuously authenticates users from their behavior
  ([U.S. Patent 11,544,358](https://patents.google.com/patent/US11544358B2/en)). Also worked on malware
  classification and adversarial robustness.

## Education and publications

- **Ph.D., Physical Chemistry**, MIT, 2018: excited-state electronic structure in organic electronics.
  Three first- or second-author papers in *J. Phys. Chem. C*
  ([Google Scholar](https://scholar.google.com/citations?user=Y4YtkbcAAAAJ)).
- **B.S., Chemistry**, *summa cum laude*, Harvey Mudd College, 2013
