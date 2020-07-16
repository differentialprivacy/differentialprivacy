---
layout: page
title: About
permalink: /about/
---

This website is intended to serve as a resource for the differential privacy research community, as well as for those seeking to learn more about the subject. If you have any comments or suggestions for topics to cover, please contact the site administrators.

## Contact

[admin@differentialprivacy.org](mailto:admin@differentialprivacy.org)


## Contributors
{% for author in site.data.contributors %}
* [{{ author[1].name }}]({{ author[1].homepage }})
{% endfor %}

## Following

<a href="https://groups.google.com/d/forum/differential-privacy-org/join"><i class="svg-icon email"></i></a>&nbsp;&nbsp;
<a href="https://www.twitter.com/DiffPriv"><i class="svg-icon twitter"></i></a>&nbsp;&nbsp;
<a href="{{ site.baseurl }}/feed.xml"><i class="svg-icon rss"></i></a>&nbsp;&nbsp;
<a href="https://github.com/differentialprivacy/differentialprivacy"><i class="svg-icon github"></i></a> 
