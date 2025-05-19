---
layout: home
title: Nuwa Improvement Proposals
---

{% capture readme_content %}{% include_relative README.md %}{% endcapture %}

{% assign fixed_content = readme_content %}

{% comment %} Fix all types of links in one go {% endcomment %}
{% assign fixed_content = fixed_content | replace: 'href="./NIPs/nip-', 'href="/nips/nip-' %}
{% assign fixed_content = fixed_content | replace: '](./NIPs/nip-', '](/nips/nip-' %}
{% assign fixed_content = fixed_content | replace: '.md"', '/"' %}
{% assign fixed_content = fixed_content | replace: '.md)', '/)' %}
{% assign fixed_content = fixed_content | replace: '[nip-template.md](./nip-template.md)', '[nip-template.md](/nip-template.md)' %}

{{ fixed_content | markdownify }}