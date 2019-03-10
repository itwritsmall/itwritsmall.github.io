---
layout: page
permalink: /tags/
title: Tags
---



{% assign tags = site.tags | sort %}

{% for tag in tags %}
  <a href="#{{ tag[0] }}">{{ tag[0] | capitalize }}</a>- {{ tag[1].size }} &nbsp;
{% endfor %}

{% for tag in tags %}
  <h3><a name="{{ tag[0] }}">{{ tag[0] | capitalize }}</a></h3>
  
  <ul>

  {% assign posts = tag[1]  | sort %}

  {% for post in posts %}
    <li><a href="{{post.url}}">{{post.title}}</a>-&nbsp;{{ post.date | date: "%m/%d/%Y" }}<br>{{ post.excerpt | strip_html | truncatewords: 20 }}
    </li>
  {%endfor%}
  
  </ul>
{% endfor %}