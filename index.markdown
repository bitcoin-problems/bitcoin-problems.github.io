---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
---

{% for prob in site.problems %}
<h2>
<a href="{{ prob.url }}"> {{prob.title}} </a>  ({{ prob.tags | join: ", " }})
</h2>
<b>status: </b> {{ prob.status }}
{{ prob.excerpt }}

{% endfor %}
