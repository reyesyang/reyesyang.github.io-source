---
layout: page
title: 文章存档
permalink: /archives/
---
<div class="page-content wc-container">
  {% for post in site.posts %}
  	{% capture currentyear %}{{post.date | date: "%Y"}}{% endcapture %}
  	{% if currentyear != year %}
    	{% unless forloop.first %}</ul>{% endunless %}
    		<h5>{{ currentyear }}</h5>
    		<ul class="posts">
    		{% capture year %}{{currentyear}}{% endcapture %}
  		{% endif %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}
</div>
