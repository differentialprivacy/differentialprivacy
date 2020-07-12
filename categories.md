---
layout: page
permalink: /categories/
title: Categories
---

<div id="archives">
{% for category in site.categories %}
  <div class="archive-group">
    {% capture category_name %}{{ category | first }}{% endcapture %}
    <div id="#{{ category_name | slugize }}"></div>
    <p></p>

    <h4 class="category-head">{{ category_name }}</h4>
    <a name="{{ category_name | slugize }}"></a>
	<ul>
    {% for post in site.categories[category_name] %}
		<li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a></li>
	{% endfor %}
	</ul>

  </div>
{% endfor %}
</div>

