---
layout: default
permalink: /
---

{% include landing.html %}

<div class="card-group mt-2">
  {% for post in site.posts %}
    {% include blog/post-card.html %}
  {% endfor %}
</div>
