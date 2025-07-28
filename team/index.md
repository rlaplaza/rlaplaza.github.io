---
title: Team
nav:
  order: 3
  tooltip: About our team
---

# {% include icon.html icon="fa-solid fa-users" %}Team


{% include section.html %}

{% include list.html data="members" component="portrait" filter="role == 'pi'" %}
{% include list.html data="members" component="portrait" filter="role != 'pi'" %}

{% include section.html background="images/background.jpg" dark=true %}

{% comment %} 

{% include section.html %}

{% capture content %}

{% include figure.html image="images/group-photo-1.jpg" %}
{% include figure.html image="images/group-photo-2.jpg" %}
{% include figure.html image="images/group-photo-3.jpg" %}

{% endcapture %}

{% endcomment %} 

{% include grid.html style="square" content=content %}
