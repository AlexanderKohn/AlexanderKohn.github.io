---
layout: default
---

## About

I'm a data scientist. I started out in physical chemistry, with a Ph.D. from MIT, and have spent the last eight
years building production ML for security and trust & safety: first at BlackBerry Cylance, and now at Blizzard
Entertainment, where I work on catching toxic chat, offensive player names and cheaters. Since 2023 a lot of that work
has involved LLMs: using them to label data, fine-tuning them, and building assistants on top of them.

A lot of my work comes down to evaluation: knowing whether a model actually does what you think it does. I'm
interested in technical AI safety, and on the side I've been running small experiments on how networks learn and on
what language models prefer. [Résumé (PDF)]({{ '/assets/Kohn_Resume_2026.pdf' | relative_url }})

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

- **[Grokking experiments](https://github.com/AlexanderKohn/grokking_experiments)** (2026): does the random draw at
  initialization decide what a small network learns when it groks? Written up [above](#writing).
- **[Preferential gaps in LLMs](https://github.com/AlexanderKohn/utility_engineering_sweetener_experiments)** (2026):
  when a model is split 50/50 between two options, is it truly indifferent, or is there a gap that a small sweetener
  can't close?

## Experience

**Blizzard Entertainment**, Senior Data Scientist, Player Interactions and Trust (2021–present)

- I own the classifiers that screen player names and in-game chat for harmful content, in games with millions of
  daily players. The current name model catches 95% of bad names at 99% precision.
- I built a chat classifier from written policy: an LLM labeled real chat against each category's definition, and
  I fine-tuned Llama 3.1 8B with LoRA on the result.
- I built the company's first internal LLM assistants and took them to production.

**BlackBerry Cylance**, Data Scientist, Research (2018–2021)

- I helped build *Persona*, a Bayesian system that keeps checking a user's identity from how they behave
  ([U.S. Patent 11,544,358](https://patents.google.com/patent/US11544358B2/en)). I also worked on malware detection
  and on making it harder to fool.

## Education and publications

- **Ph.D., Physical Chemistry**, MIT, 2018. I modeled how organic molecules lose energy from excited states, which
  led to three papers in *J. Phys. Chem. C* ([Google Scholar](https://scholar.google.com/citations?user=Y4YtkbcAAAAJ)).
- **B.S., Chemistry**, *summa cum laude*, Harvey Mudd College, 2013
