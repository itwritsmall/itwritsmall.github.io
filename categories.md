---
layout: page
permalink: /categories/
title: Categories
---




{% assign categories = site.categories | sort %}

{% for category in categories %}
  <a href="#{{ category[0] }}">{{ category[0] | capitalize }}</a>- {{ category[1].size }} &nbsp;
{% endfor %}

{% for category in categories %}
  <h3><a name="{{ category[0] }}">{{ category[0] | capitalize }}</a></h3>
  
  <ul>

  {% assign posts = category[1]  | sort %}

  {% for post in posts %}
    <li><a href="{{post.url}}">{{post.title}}</a>-&nbsp;{{ post.date | date: "%m/%d/%Y" }}<br>{{ post.excerpt | strip_html | truncatewords: 20 }}
    </li>
  {%endfor%}
  
  </ul>
{% endfor %}