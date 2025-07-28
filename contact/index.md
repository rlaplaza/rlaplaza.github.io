---
title: Contact
nav:
  order: 5
  tooltip: Email, address, and location
---

# {% include icon.html icon="fa-regular fa-envelope" %}Contact

Reach out with any inquiries you may have!

{%
  include button.html
  type="email"
  text="Ruben's email"
  link="ruben.laplaza@iiq.csic.es"
%}
{% comment %}
{%
  include button.html
  type="phone"
  text="(555) 867-5309"
  link="+1-555-867-5309"
%}
{% endcomment %}
{%
  include button.html
  type="address"
  tooltip="Our location on Google Maps for easy navigation"
  link="https://maps.app.goo.gl/qFEJceqGLHkFQxPk6"
%}

{% include section.html %}

{% capture col1 %}

{%
  include figure.html
  image="images/photo-location-1.jpg"
  caption="The institute"
%}

{% endcapture %}

{% capture col2 %}

{%
  include figure.html
  image="images/photo-location-2.jpg"
  caption="The main entrance"
%}

{% endcapture %}

{% include cols.html col1=col1 col2=col2 %}

{% comment %}
{% include section.html dark=true %}

{% capture col1 %}
Lorem ipsum dolor sit amet  
consectetur adipiscing elit  
sed do eiusmod tempor
{% endcapture %}

{% capture col2 %}
Lorem ipsum dolor sit amet  
consectetur adipiscing elit  
sed do eiusmod tempor
{% endcapture %}

{% capture col3 %}
Lorem ipsum dolor sit amet  
consectetur adipiscing elit  
sed do eiusmod tempor
{% endcapture %}

{% include cols.html col1=col1 col2=col2 col3=col3 %}
{% endcomment %}
