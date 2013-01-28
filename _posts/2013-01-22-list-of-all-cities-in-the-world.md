---
layout: post
title: List of all cities in the world
location: Waterloo, Canada
comments: true
tags:
 - geoip
 - csv
 - database
 - parser
 - python
---

Hello internets. If you're looking for a list of all cities in the world,
you found it.

It turns out that my web service <http://freegeoip.net> has a database
of IP addresses, and columns like country, region and city names. The region
column is the only one where names are not properly spelled, they are missing
the accents. e.g.: Sao Paulo instead of São Paulo.

So while searching the web for a list of all regions in the world (by the
way, regions are usually states or provinces of a country) google gave
me this: <http://answers.google.com/answers/threadview/id/774429.html>

It has a flat file database with all cities, towns, administrative
divisions and agglomerations with their population. The post dates
from 2006, but I'm only interested in region names and they (hopefully)
haven't drastically changed since then.

With a bit of programming it was possible to extract only the necessary
information and compile this list of country, region and city in a CSV file,
encoded as UTF-8.

<hr>

Download: <http://musta.sh/files/all_cities_in_the_world.csv.zip>

<hr>

This is the script used to compile the list:

<script src="https://gist.github.com/4592774.js"> </script>

Now it's time to normalize all region names in the GeoIP database. The idea
is to match region names from the new list against these ones:
<http://dev.maxmind.com/static/csv/codes/maxmind/region.csv>
