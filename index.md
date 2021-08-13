## You need to hide API key
You need to hide API key when using google maps js API. It is not enough for you to set the referer.
I have local web server on my PC and can change my hosts file, So that I can spoof domain name of my HTML as your domain.
If you reveal your API key in your HTML, someone might access Google map with that key. This could mean a million page refreshes (and map loads)!

This is bad example from Google.

```
<script defer
  src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&callback=initMap">
</script>
```
## How to hide API key from HTML
I use Environment Variables and CGI to hide my API key from HTML as follows.

**1. set Environment Variables**

I set Google Maps API key in Environment Variables and passing it to my CGI scripts.
nginx + fcgiwrap are running on my server, so I set API key in my fcgiwrap.conf like this.

fcgiwrap.conf
```
location /cgi-bin/ {
    ........
    fastcgi_param GOOGLE_MAPS_API_KEY　 YOUR_API_KEY; <= SET YOUR API KEY HERE
}
```
**2. make CGI script**

I made python CGI like this. This is same as `src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY` in SAMPLE of Google.

getapijs.py
```
#!/usr/bin/python
# -*- coding: utf-8 -*-
import requests
import os
url = 'https://maps.googleapis.com/maps/api/js'
key = os.environ['GOOGLE_MAPS_API_KEY'] # get Environment Variables
mysrc = url + "?key=" + key
response = requests.get(mysrc) # get src from Google Maps url
print("'Content-Type': 'text/javascript; charset=UTF-8'") # header for HTML
print("")
print(response.text)
```

**3. call CGI from javascript**

call your CGI from window.onload. This is same as `<script defer ... &callback=initMap>` in SAMPLE from Google.

main.js
```
function initMap() {
    var uluru = {lat: -25.344, lng: 131.036};
    var map = new google.maps.Map(document.getElementById('map'), {zoom: 4, center: uluru});
    var marker = new google.maps.Marker({position: uluru, map: map});
}
window.onload = function() {
    fetch("/cgi-bin/getapijs.py").then(res=>{
        return res.text();
    }).then(mytext => {
        eval(mytext);
    }).then(() => {
        initMap();
    }).catch(() =>{
        // error handling
    });
}
```
**4.read main.js in your HTML**

set `<script src="main.js"></script>` in your header section.

index.html
```
<!DOCTYPE html>
<html>
  <head>
    <style>
      /* Set the size of the div element that contains the map */
      #map {
        height: 400px;  /* The height is 400 pixels */
        width: 100%;  /* The width is the width of the web page */
      }
    </style>
    <title>Hello World</title>
    <script src="main.js"></script>
  </head>
  <body>
    <h3>My Google Maps Demo</h3>
    <!--The div element for the map -->
    <div id="map"></div>
  </body>
</html>
```



**Plese visit our website.**
### [WEB System Infrastructure Guide for Beginners](https://www.olto3-sugi3.tk/index.html)
