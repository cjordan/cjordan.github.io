---
title: Blog
layout: default
---

{% for post in site.posts %}
  <div class="panel panel-primary">
    <div class="panel-heading">
      <a href="{{ post.url }}" style="color: #fff">
        <h3 class="panel-title">{{ post.title }}</h3>
      </a>
    </div>

    <div class="panel-body">
      {{ post.content | strip_html | truncate: 500 }}
    </div>

    <div class="panel-footer">
      <a href="{{ post.url }}#disqus_thread" data-disqus-identifier="{{ post.url }}"></a>
    </div>
  </div>
{% endfor %}
