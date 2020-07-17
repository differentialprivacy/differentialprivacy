---
layout: page
title: About
permalink: /about/
---

This website is intended to serve as a resource for the differential privacy research community, as well as for those seeking to learn more about the subject. If you have any comments or suggestions for topics to cover, please contact the site administrators.

## Contact

<admin@differentialprivacy.org>

## Mailing list

Please [join our mailing list](https://groups.google.com/d/forum/differential-privacy-org/join) for updates.

## Contributors
{% for author in site.data.contributors %}
* [{{ author[1].name }}]({{ author[1].homepage }})
{% endfor %}

## Subscribe and Follow

<a href="https://feedburner.google.com/fb/a/mailverify?uri=DifferentialPrivacy"><i class="svg-icon email"></i></a>&nbsp;&nbsp;
<a href="https://www.twitter.com/DiffPriv"><i class="svg-icon twitter"></i></a>&nbsp;&nbsp;
<a href="{{ site.baseurl }}/feed.xml"><i class="svg-icon rss"></i></a>&nbsp;&nbsp;
<a href="https://github.com/differentialprivacy/differentialprivacy"><i class="svg-icon github"></i></a> 

<form style="border:1px solid #ccc;padding:3px;text-align:center;" action="https://feedburner.google.com/fb/a/mailverify" method="post" target="popupwindow" onsubmit="window.open('https://feedburner.google.com/fb/a/mailverify?uri=DifferentialPrivacy', 'popupwindow', 'scrollbars=yes,width=550,height=520');return true"><p>Enter your email address:</p><p><input type="text" style="width:140px" name="email"/></p><input type="hidden" value="DifferentialPrivacy" name="uri"/><input type="hidden" name="loc" value="en_US"/><input type="submit" value="Subscribe" /><p>Delivered by <a href="https://feedburner.google.com" target="_blank">FeedBurner</a></p></form>
