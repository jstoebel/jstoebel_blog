---
layout: post
title: "Using react leaflet"
permalink: using-react-leaflet
date: 2017-08-10 15:10:57
comments: true
description: "Using react-leaflet"
keywords: ""
categories:

tags:

---

For my new [nightlife app](https://github.com/jstoebel/nightlife) I am using [`leaflet.js`](http://leafletjs.com/) to handle marking search results on a map. Leaflet was an immediate win for this project because so much comes, basically for free right out of the box: you get beautiful looking maps, markers, and pop ups with hardly anything to configure. What could be better? Oh I know, how about wrapping this functionality in React components! That way I can render a series of markers on a map with something like:

{% highlight jsx %}
<Map bounds={this.state.corners}>
  <TileLayer
    url='http://{s}.tile.osm.org/{z}/{x}/{y}.png'
    attribution='&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors'
  />
  {this.props.results.map(this._eachBar)}
</Map>  
{% endhighlight %}

Neato! There are a few gotchas with `react-leaflet` though. At its core, `leaflet` is not using React, and instead manipulates the DOM on its own. This creates some interesting, unexpected results. For example when a popup is rendered, its doesn't go where you expect it to go in the virtual DOM:

{% highlight jsx%}
<BrowserRouter></BrowserRouter>
<div>
    <h5> Burgers and Brew </h5>
    </rsvpButton>
</div>
{% endhighlight %}

Doh! This means that the popup is completely outside the of the context and thus doesn't have access to the redux store. If you were hoping to `mapStateToProps` like normal, you're out of luck. Instead we have to pass props in the old fashioned way:

{% highlight jsx %}
/* 
  Map.jsx
  map component. This function shows representation of a single bar which 
  contains a marker with popup. Since passing in state won't work, we just
  pass the props in directly. 
*/

_eachBar(bar, idx) {
  // render a single bar on the map

  //grab the cooridnates in strucutre leaflet requires
  const coords = [
    bar.coordinates.latitude,
    bar.coordinates.longitude
  ]
  return (
    <Marker position={coords} key={idx} >
      <Popup>
        <div>
          <h5>{bar.name}</h5>
          <RSVPButton 
            bar={bar} 
            currentRSVPs={this.props.currentRSVPs} 
            onFetchBars={this.props.onFetchBars}
            onAddError={this.props.onAddError}
          />
        </div>
      </Popup>
    </Marker>
  )
} 
{% endhighlight %}

I've only begun to scratch the surface with what leaflet can do. Its not implemented with React so at its core you need to be a little flexible with your style, but in the end I think its worth working with.