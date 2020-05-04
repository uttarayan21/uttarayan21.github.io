---
layout: default
---

This is my "blog".

Just here to write what I feel like. Maybe ? IDK.

Stuff I wrote :- 

{% for post in site.posts %}
  <article>
    <h2>
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h2>
    <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time>
  </article>
{% endfor %}

