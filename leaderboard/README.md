Leaderboard Visualization
=============

The Leaderboard Visualization needs to show the current leaders in both game play objectives.

First, a leaderboard for the highest barscore. We figured the more people frequented the bar the more drinks they had to be consuming.  Therefore the complicated mathematical equation of:

> high barscore = high level of drunkenness

The second game objective would be to claim all of the beacons hidden in the club. The party had somewhat of a TRON related theme, so to visualize each beacon we borrowed some nerdtastic TRON symbols.

For both leaderboards wanted to incorporate the players’ pictures and their screen names and add a little animation into the mix.

And here is what the end result looks like:

![alt text](/img/visualization-banner.jpg "Leaderboard Visualization")

### Fixed Screen Size and Other Configurations

Again, similar to the ['People At The Bar' Visualization](/bar/) I decided to implement this with a fixed screen size of 1900x1060 resolution using Mike Bostock's [margin convention](http://bl.ocks.org/mbostock/3019563).

Looking at the source code you will notice several variables being set at the top. Because both leaderboards are being shown on the same screen , some of these configurations are shared, others specific to each visualization.

All of these should be quite self explanatory after reading the comments.

```javascript
		
		// SHARED vars
		var svg, // the svg canvas
			leaderCount = 8, // number of leaders shown
			refreshTime = 7000, // milliseconds between data refresh requests
			vizWidth = 1900, // total screen width
			vizHeight = 1060, // total screen height
			margin = {top: 80, right: 90, bottom: 60, left: 330}, // the margins of each leaderboard
			padding = 13, 
			animDuration = 2000, // the time it takes to animate a change
			width =  vizWidth/2 - margin.left - margin.right, 
			height = vizHeight - margin.top - margin.bottom, // individual leaderboard height
			radius = ( height - padding*leaderCount ) / ( leaderCount*2 ), // radious of the player selfie
			picFrameWidth = 2;
		
		// BEACON vars	
		var beaconViz; // the svg group containing the beacons leaderboard components
			
		// BARSCORE vars
		var bsViz, // the svg group containing the beacons leaderboard components
			barHeight = 30, // the height of each barscore bar
			minBarWidth = radius + padding + 10, // the minimum width of a bar
			bsx = d3.scale.linear().range([0, width-minBarWidth]); // the x-scale calculating the bar width

```

### Real-time Server Updates

I implemented this with simple polling using 'setInterval()' and the 'd3.json() method refreshing the information shown on the screen in intervals defined by 'refreshTime' (7000 milliseconds in our case).

The callback then calls `drawBarScoreLeaders()' and 'drawBeaconLeaders()' passing the newly loaded data as an argument.

```javascript

		// refresh the data
		setInterval(function() {

			d3.json("http://ec2-54-200-209-241.us-west-2.compute.amazonaws.com/leaderboard/" + leaderCount , function(error, data) {

				drawBarScoreLeaders(data);
				drawBeaconLeaders(data);

			});

		}, refreshTime);

```

### Binding DOM Elements to Data with D3js

Before you read on it will be helpful if you have a general understanding of binding DOM elements to data using D3js.

If you are not familiar with this please take a look at:

[D3 selection](https://github.com/mbostock/d3/wiki/Selections)

[D3 selection.data](https://github.com/mbostock/d3/wiki/Selections#data)


## The Barscore Leaderboard

The Barscore Leaderboard is basically a bar chart with screen name labels and a players' picture on the left and barscores displayed on the right. Leading players are shown on the top, new leaderboard arrivals animate onto the screen from above pushing less successful players off the screen, position changes would animate respectively.






## The Beacon Leaderboard

The Beacon Leaderboard would simply show the players who were most successful at collecting beacons and highlight the ones they had claimed.

Looking at the source code you will find `drawBeaconLeaders()` which expects the `data` argument containg the most recent group of barscore leaders.

```javascript 

/**
 * Draws and updates the BEACON claim leaderboard
 */
function drawBeaconLeaders(data){

	var beaconLeaders = data.claimLeaders,
		symbolStartPos = radius + 20,
		symbolSpacing = (width-25) / 6,
		symbolSize = 80,
		rows,
		row;
	
	// bind data to DOM elements
	rows = beaconViz.selectAll(".beacon-leader")
			.data(beaconLeaders, function(d) { return d.USER_ID; });

```

#### Constructing New Leaders

First create a svg group that will be our container for all the elements. This is convenient as we can assign a css class, 'beacon-leader' in this case, for applying styles conveniently.

```javascript 
	
	// CREATE a node for each new leader
	row = rows.enter()
			.append("g")
			.attr("class","beacon-leader");

```
Then create a screen name label and 2 circles creating the frame for the player picture

```javascript
	// name label on left
	row.append("text")
		.attr("class", "name")
		.attr("y", 0)
		.attr("x", -radius - padding)
		.attr("dy", ".35em");

	// add photo outer frame
	row.append("circle")
		.attr("class", "outerCircle")
		.attr("r", radius + padding)
		.attr("cx", 0)
		.attr("cy", 0);

	// add photo frame
	row.append("circle")
		.attr("class", "picFrame")
		.attr("r", radius + picFrameWidth) 
		.attr("cx", 0)
		.attr("cy", 0);


```



