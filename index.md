---
layout: home
title: Alexander Kohn
home: true
---

## About

I'm a data scientist at Blizzard Entertainment, where I work on catching toxic chat, offensive player names and
cheaters. I trained as a physical chemist (Ph.D., MIT) and have spent the eight years since building production ML
for security and trust & safety, first at BlackBerry Cylance and now at Blizzard. Since 2023 a lot of that work has
involved LLMs: using them to label data, fine-tuning them, and building assistants on top of them.

Much of the job is evaluation, which means finding out whether a model actually does what you think it does. I'm
interested in technical AI safety, and on the side I run small experiments on how networks learn and on what
language models prefer.

## Writing

<ul class="entries">
{% for post in site.posts %}
  <li>
    <strong><a href="{{ post.url | relative_url }}">{{ post.title }}</a></strong>
    <span class="date">{{ post.date | date: "%B %Y" }}</span><br>
    {{ post.description }}
  </li>
{% endfor %}
</ul>

## Side projects

<ul class="entries">
  <li>
    <strong><a href="https://github.com/AlexanderKohn/grokking_experiments">Grokking experiments</a></strong>
    <span class="date">2026</span><br>
    The code, the plan and the full results behind the post above.
  </li>
  <li>
    <strong><a href="https://github.com/AlexanderKohn/utility_engineering_sweetener_experiments">Preferential gaps in LLMs</a></strong>
    <span class="date">2026</span><br>
    When a model is split 50/50 between two options, is it truly indifferent, or is there a gap that a small
    sweetener can't close?
  </li>
</ul>

## Experience

### Blizzard Entertainment

Senior Data Scientist, Player Interactions and Trust · 2021–present
{: .role}

- I own the classifiers that screen player names and in-game chat for harmful content, in games with millions of
  daily players. The current name model catches 95% of bad names at 99% precision.
- I built a chat classifier from written policy. An LLM labeled real chat against each category's definition, and
  I fine-tuned Llama 3.1 8B with LoRA on the result.
- I built the company's first internal LLM assistants and took them to production.

### BlackBerry Cylance

Data Scientist, Research · 2018–2021
{: .role}

- I helped build *Persona*, a Bayesian system that keeps checking a user's identity from how they behave
  ([U.S. Patent 11,544,358](https://patents.google.com/patent/US11544358B2/en)). I also worked on malware detection
  and on making it harder to fool.

## Education

### MIT

Ph.D., Physical Chemistry · 2018
{: .role}

I modeled how organic molecules lose energy from excited states, which led to three papers in *J. Phys. Chem. C*
([Google Scholar](https://scholar.google.com/citations?user=Y4YtkbcAAAAJ)).

### Harvey Mudd College

B.S., Chemistry, *summa cum laude* · 2013
{: .role}
