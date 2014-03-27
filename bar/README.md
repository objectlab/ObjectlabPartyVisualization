### The 'At The Bar' Visualization

We figured the best way to visualize micro-location detection was to simulate typical 'people at the bar' behaviour, to display their mugshot and screen name and attempt to engage them with game related personalized messages. All we needed was an interesting way to mimick people bumping into each other, trying to push through the crowd to order a drink and having a good time.

And here is what we came up with:

![alt text](/img/people-at-bar.jpg "At The Bar Visualization")

People at the bar are visualized as bubbles floating on the screen. New arrivals come bouncing onto the screen from above causing movement and eventually settling down until somebody else arrives. They are greeted with a personalized message displayed as a call-out offering game playing tips or congratulating them on their performance. As more people crowd the space and bump into each other more movement happens… you know, kind of typical bar behaviour.

### Fixed Screen Size

- Explain fixed screen size


### Constructing a Person

Describe how each bubble is built


### The Animated Layout

To auto-position and animate the people I am using [d3.layout.force](https://github.com/mbostock/d3/wiki/Force-Layout#force) which constructs a force-directed layout and does all of the complicated math for us to simulate attraction/repulsion, friction, proximity, charge and gravity along with others. These are all the ingredients needed to accomplish what we need.

```javascript
// configure the force layout
force = d3.layout.force()
	    .nodes(peopleOnScreen)
	    .size([width, height])
	    .gravity(0.005)
	    .charge(-radius*3.5)
	    .on("tick", onForceTick)
	    .start();
```

### Getting 'Live' Updates from the Server

- Describe how to poll the server
- Describe how to establish the delta


### Newcomers, Updates and Departures

- Why we can simply apply newly loaded data 




