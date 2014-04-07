Leaderboard Visualization
=============

The Leaderboard Visualization needs to show the current leaders in both game play objectives.

#####Objective 1: Highest Barscore
We figured the more people frequented the bar the more drinks they had to be consuming.  Therefore the complicated mathematical equation of:

> high barscore = high level of drunkenness


#####Objective 2: Claim All Beacons
The second game objective would be to claim all of the beacons hidden in the club. The party had somewhat of a TRON related theme, so to visualize each beacon we borrowed some nerdtastic TRON symbols.


For both leaderboards we wanted to incorporate the players’ pictures and their screen names and add a little animation into the mix.

And here is what the end result looks like:

![alt text](/img/visualization-banner.jpg "Leaderboard Visualization")

### Fixed Screen Size and Other Configurations

Again, similar to the ['People At The Bar' Visualization](/bar/) we decided to implement this with a fixed screen size of 1900x1060 resolution using Mike Bostock's [margin convention](http://bl.ocks.org/mbostock/3019563).

Looking at the source code you will notice several variables being set at the top. Because both leaderboards are being shown on the same screen, some of these configurations are shared, others are specific to each visualization.

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

We implemented this with simple client-server polling using `setInterval()` and the `d3.json()` method refreshing the information shown on the screen in intervals set by `refreshTime` (7000 milliseconds in our case).

The callback defined for `d3.json()` then calls `drawBarScoreLeaders()` and `drawBeaconLeaders()` passing the newly loaded data as an argument.

```javascript

		// refresh the data
		setInterval(function() {

			d3.json("http://Your/Server/leaderboard/" + leaderCount , function(error, data) {

				drawBarScoreLeaders(data);
				drawBeaconLeaders(data);

			});

		}, refreshTime);

```

### Binding DOM Elements to Data with D3js

Before you read on it will be helpful if you have a general understanding of binding DOM elements to data using D3js.

If you are not familiar with, this please take a look at:

[D3 selection](https://github.com/mbostock/d3/wiki/Selections)

[D3 selection.data](https://github.com/mbostock/d3/wiki/Selections#data)


## The Barscore Leaderboard

The Barscore Leaderboard is basically a bar chart with screen name labels and a players' picture on the left and barscores displayed on the right. Leading players are shown on the top, new leaderboard arrivals animate onto the screen from above pushing less successful players off the screen, position changes would animate respectively.

Looking at the source code you will find `drawBarScoreLeaders()` which expects a `data` argument containg the most recent group of barscore leaders in the `data.scoreLeaders` array.

As barscores increase we need to continue to re-calibrate the x-scale that calculates the width for each bar so they will not grow off the screen. This is done by resetting the [d3.scale.domain](https://github.com/mbostock/d3/wiki/Quantitative-Scales#linear_domain) using [d3.extent](https://github.com/mbostock/d3/wiki/Arrays#d3_extent) to figure out the min and max barscore values.

```javascript 

function drawBarScoreLeaders(data){

		var bsLeaders = data.scoreLeaders,
			bars,
			bar;
		
		// calibrate the the bar width x-scale with newly loaded max barscore values
		bsx.domain(d3.extent(bsLeaders, function(d) { return d.BAR_SCORE; })).nice();
		
```

Then we bind existing DOM elements to the newly loaded data.


```javascript
		// bind existing DOM elements for leaders to new data
		bars = bsViz.selectAll(".bs-leader")
				.data(bsLeaders, function(d) { return d.USER_ID; });
```


##### Constructing New Barscore Leaders

First create a svg group that will be our container for all the elements. This is convenient as we can assign a css class, 'beacon-leader' in this case, for applying styles conveniently.

```javascript

		// CREATE nodes for each leader
		bar = bars.enter()
				.append("g")
				.attr("class","bs-leader");

```

Now create a svg rectangle for our barscore 'bar', a screen name label positioned on the left, a barscore label positioned on the right and 2 circles creating the frame for the player picture.

```javascript
		// the bar
		bar.append("rect")
			.attr("width", function(d) {
				return bsx(d.BAR_SCORE) + minBarWidth;
			})
			.attr("height", barHeight)
			.attr("x", 0)
			.attr("y", -barHeight/2);
		
		
		// score label on right
		bar.append("text")
			.attr("class", "score")
			.attr("y", 0)
			.attr("dy", ".35em");
		
		// name label on left
		bar.append("text")
			.attr("class", "name")
			.attr("y", 0)
			.attr("x", -radius - padding)
			.attr("dy", ".35em");
```

And finally add the player picture

```javascript
		// add the pic
		bar.append("image")
			.attr("class","photo")
			.attr("xlink:href", function(d) { return imgUrl(d); })
			.attr("x", -radius)
      		.attr("y", -radius - radius/2 )
			.attr("width", radius*2)
      		.attr("height", radius*1.5*2);	

```

#####Remove players who have been bested by others

For this we use  [selection.exit()](https://github.com/mbostock/d3/wiki/Selections#exit) and simply remove bars that are not needed.

```javascript

		// DELETE un-needed leader nodes
		bars.exit().remove();

```

#####Update Extisting Leaders

For players who are already displayed from a previous data load we need to update their vertical position, the width of the 'bar', the barscore label, their screen name and their photo.

```javascript

		// UPDATE existing leader nodes
		bars.transition().duration(animDuration).ease("exp-out")
			.call(positionLeader);
	
			
		bars.select("rect").transition().duration(animDuration).ease("exp-out")
			.attr("width", function(d) {
				return bsx(d.BAR_SCORE) + minBarWidth;
			});
		
		// UPDATE score
		bars.select(".score")
			.text(function(d) {
				return d.BAR_SCORE;
			})
			.transition().duration(animDuration).ease("exp-out")
				.attr("x", function(d) {
					return bsx(d.BAR_SCORE) + minBarWidth + padding/2;
				});
				
		// UPDATE name	
		bars.select(".name")
			.text(function(d, i) {
				return (i+1) + '.' + d.NAME;
			});
			
		// UPDATE photo
		bars.select(".photo")
		    .attr("xlink:href", function(d) { return imgUrl(d); });

```





## The Beacon Leaderboard

The Beacon Leaderboard shows the players who were most successful at collecting beacons and highlight the ones they had claimed.

Looking at the source code you will find `drawBeaconLeaders()` which expects the `data` argument containg the most recent group of barscore leaders. `data.claimLeaders` is an Array containing our beacon claim leaders.

Our first task is to again bind DOM elements for existing players with the newly loaded data. 

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

##### Constructing New Leaders

Again we create a svg group that will be our container for all the elements and assign a css class, 'beacon-leader' in this case.

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



