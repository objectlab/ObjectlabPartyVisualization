### The 'People At The Bar' Visualization

We figured the best way to visualize micro-location detection was to simulate typical 'people at the bar' behaviour, to display their mugshot and screen name and attempt to engage them with game related personalized messages. All we needed was an interesting way to mimick people bumping into each other, trying to push through the crowd to order a drink and having a good time.

And here is what we came up with:

![alt text](/img/people-at-bar.jpg "At The Bar Visualization")

People at the bar are visualized as bubbles floating on the screen. New arrivals come bouncing onto the screen from above causing movement and eventually settling down until somebody else arrives. They are greeted with a personalized message displayed as a call-out offering game playing tips or congratulating them on their performance. As more people crowd the space and bump into each other more movement happens… you know, kind of typical bar behaviour.

### Fixed Screen Size and Other Settings

To keep things as simple I decided to implement this with a fixed screen sizee. Our A/V guys provided screens with 1900x1060 resolution so that is what I used. Looking at the source code you will notice several variables being set at the top:

```javascript
// CONFIGS
    var svg, // the svg canvas
	barViz, // the svg group containing the bar visualization
	barCount = 10, // the max amount of people-bubble on the screen 
	refreshTime = 7000, // milliseconds between data refresh requests 
	vizWidth = 1900, //1920, // total screen width
	vizHeight = 1060, //1080, // total screen height
	margin = {top: 30, right: 30, bottom: 30, left: 30},
	padding = 13, // padding between people-bubbles
	animDuration = 2000, // the time it takes to animate a change
	width =  vizWidth - margin.left - margin.right, // width of visualization
	height = vizHeight - margin.top - margin.bottom, // height of visualization
	radius = 100, // bubble radius
	radiusBig = 200, // new arrival bubble radius
	textPadding = 22,
	picFrameWidth = 2,
	lastLoadedPeople = [],
	peopleOnScreen = [],
	force;
```





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




