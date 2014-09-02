---
layout: page
title: Articles
excerpt: "An archive of articles sorted by date."
image:
  feature: vrindavan-gobardhan-yatra-02.jpg
  credit: me :)
  creditlink: https://www.flickr.com/photos/95959458@N02/14929455577/
  avatar: sahilsk.jpg
---

<ul class="post-list">
{% for post in site.categories.articles %}
  <li><article><a href="{{ site.url }}{{ post.url }}">{{ post.title }} <span class="entry-date"><time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time></span></a></article></li>
{% endfor %}
</ul>
