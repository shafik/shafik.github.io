---
layout: default
---

{% include head.html %}

{% if site.twitter_username %}
  <li>
    <a href="https://twitter.com/{{ site.twitter_username }}">
      <i class="fa fa-twitter"></i> Twitter
    </a>
{% endif %}

{% if site.stackoverflow_id %}
    <a href="https://stackoverflow.com/users/{{ site.stackoverflow_id }}">
      <i class="fa fa-stack-overflow"></i> StackOverflow 
    </a>
  </li>
{% endif %}

## Blog posts 

<ul>
  {% for post in site.posts %}
  {% if post.categories contains "polls" %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endif %}
  {% endfor %}
</ul>
