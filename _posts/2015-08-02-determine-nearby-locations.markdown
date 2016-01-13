---
layout: post
title:  "How to determine nearby locations with trigonometry!"
date:   2015-07-15 22:46:00
description: Learn how to determine locations nearby using latitudes & longitudes, a radius and your location data
# categories: blog geolocation geofencing latitude longitude
tags: geolocation geofencing math trigonometry sql
thumbnail: /images/angle.png
---

Lots of apps out there figure out how to determine locations near you. So just how exactly do we do that? Well SQLServer has some nice functionality baked right in. So does Android and iOS. What if you're using mySQL and all you've got are latitudes and longitudes? How would you figure out which of those locations are near where you are?

Do you remember your high school trigonometry? Does the acronym `SOHCAHTOA` make you cringe? If you’ve answered yes to either of these questions and have bad flashbacks or the urge to break out your TI-92 calculator don’t fret. If you've no idea what I'm talking about you must be one of the cool kids. Either way, we’re going to review some of the basics. 

`SOHCAHTOA` is a mnemonic to help you remember your trigonometry. The following equations define the relationships between the sides and angles of triangles. We're not going to give a complete trigonometry refresher here but this should jog your memory a little bit.

![SOHCAHTOA Explained]({{ site.baseurl }}/images/angle.png){: .img-responsive .center-block }

{% highlight matlab%}
 sin(θ) = opposite/hypotenuse (SOH) 
 cos(θ) = adjecent/hypotenuse (CAH)
 tan(θ) = opposite/adjecent (TOA)
{% endhighlight %} 

Despite what some may believe the earth is round. Pictures taken from the Apollo program's command module prove the earth is round... well ok to be technical it's more like an oblate spheroid if we're going to be precise. 

![Yep, the earth is round!]({{ site.baseurl }}/images/earth.jpg){: .img-responsive .center-block }

With that in mind we need to understand that there is a system by which any point on the surface of a round object, such as the Earth, can be found using a geographic coordinate system. This system uses a grid that's laid over of the earth. This grid of imaginary lines, connect the north pole to the south pole or the top of the Earth to the bottom. These are called meridians and represent all 360 degrees ranging from -180 to 180 where 0 represents the prime meridian. The imaginary lines that circle around the earth are parallels. With the prime parallel located at the equator parallels in the southern hemisphere range from 0 degrees to -90 degrees and in the northern hemisphere range from 0 to 90 degrees. This coordinate system is what your car’s GPS uses to tell you where you are and help you get to where you’re going.

Longitude measures the meridians and latitude measures the parallels. With latitude and longitude you can determine any spot on the surface of the earth. With these being measure in degrees we can use some trigonometry to determine the distance between two points. There’s a super neat formula called the [spherical law of cosines](http://www.movable-type.co.uk/scripts/latlong.html). 


Given the following table where `LATITUDE` and `LONGITUDE` are columns that contain latitude and longitude coordinate data:

{% highlight sql %}
CREATE TABLE LOCATIONS
(
DESCRIPTION varchar(100),
LATITUDE float,
LONGITUDE float
);
{% endhighlight %}

So let's say we wanted to find locations near the Empire State Building in New York City. The Empire State Building is located at `40.7484405,-73.9856644`

<iframe src="https://www.google.com/maps/embed?pb=!1m14!1m8!1m3!1d3022.6175398570613!2d-73.9856644!3d40.7484405!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x89c259a9b3117469%3A0xd134e199a405a163!2sEmpire+State+Building!5e0!3m2!1sen!2sus!4v1438532564350" height="450" frameborder="0" style="width:100% !important;border:0;display:block;" allowfullscreen></iframe>

We can then use the following SQL statement to see all locations that are within 10 miles of `40.7484405,-73.9856644`

{% highlight java %}
// RADIUS OF EARTH IN MILES is 3963.1676
// SEARCH RADIUS IN MILES is 10
SELECT * 
FROM LOCATIONS
WHERE ACOS( SIN( RADIANS( LATITUDE ) ) * SIN( RADIANS( 40.7484405 ) ) + 
 COS( RADIANS ( LATITUDE ) ) * COS( RADIANS( 40.7484405 ) ) * COS( RADIANS( -73.9856644 ) - RADIANS( LONGITUDE ) ) ) * 3963.1676 < 10
{% endhighlight %}

And there you have it. This SQL Select statement would pull back all records from a locations table that are less than 10 miles from your latitude and longitude `40.7484405,-73.9856644`. This could also be adapted for Excel spreadsheets or your own database table of location data. Just keep in mind that you'll need to convert your latitude and longitude from degrees to radians and that the radius of the earth, in miles, is 3963.1676 miles. 

Hope this helps!
- Mike
