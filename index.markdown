---
layout: default
title: "Who Care !"
description: "Personal blog of Hexa Lee"
---
{% for page in site.posts limit:5 %}
{% assign body = page.content %}
{% include post-div.html %}
{% endfor %}
<div class="related">
<h3>More Posts</h3>
<p>{% for post in site.posts offset:5 %}<a href="{{ post.url }}">{{ post.title }}</a>{% unless forloop.last %} &middot; {% endunless %}{% endfor %}</p>
</div>