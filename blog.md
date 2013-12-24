---
title: Blog
layout: default
---

{% for post in site.posts %}
<div class="well" style="padding-top: 0px">
    <a href="{{ post.url }}">
        <h2>{{ post.title }}</h2>
    </a>
        {{ post.content | strip_html | truncate: 500 }}
</div>
{% endfor %}
