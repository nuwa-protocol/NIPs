---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

{% capture readme_content %}{% include_relative README.md %}{% endcapture %}
{{ readme_content | markdownify }}