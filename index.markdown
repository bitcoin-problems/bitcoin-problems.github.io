---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

# About

*Bitcoin Problems* is a community managed list of open research problems that, if solved, would have positive impact on the evolution of Bitcoin.
Its purpose to help align the efforts of researchers and the needs of Bitcoin protocol developers.

----
# Open Problems

{% for prob in site.problems %}
<h2>
<a href="{{ prob.url }}"> {{prob.title}} </a>  <small>({{ prob.tags | join: ", " }})</small>
</h2>
<b>status: </b> {{ prob.status }}
{{ prob.excerpt }}

{% endfor %}


[Lightning Network]: https://en.wikipedia.org/wiki/Lightning_Network
