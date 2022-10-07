---
layout: navbar
title: Othman's Articles
---

## Sysadmin

My technical IT articles for different technologies.

<ul class="horizontal-list">
{% for post in site.posts %}
    {% if post.category == "sysadmin" %}
        {% include card.html %}
    {% endif %}
{% endfor %}
</ul>


## Translations

My translations of technical IT articles from English to Arabic.

<ul class="horizontal-list">
{% for post in site.posts %}
    {% if post.category == "translation" %}
        {% include translation.html %}
    {% endif %}
{% endfor %}
</ul>


## All Articles
<ul class="horizontal-list">
{% for post in site.posts %}
    {% include post.html %}
{% endfor %}
</ul>
