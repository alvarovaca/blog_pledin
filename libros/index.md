---
layout: single
permalink: /libros/index.html
author_profile: true
classes: wide
sidebar:
  nav: "all"
comments: true
---
# Colección de Libros Libres

{% for categoria in site.data.libros %}
## [{{categoria.categoria}}]({{categoria.url}})

{% for libro in categoria.libros %}
* {{libro.nombre}} [[web]({{libro.web}})] [[pdf]({{libro.url}})]
{% endfor %}
{% endfor %}

