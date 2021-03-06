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


## 1.The Barscore Leaderboard

The Barscore Leaderboard is basically a horizontal bar chart with screen name labels and a players' picture on the left and barscores displayed on the right. Leading players are shown on the top, new leaderboard arrivals animate onto the screen from above pushing less successful players off the screen; position changes animate respectively.

Looking at the source code you will find `drawBarScoreLeaders()` which expects a `data` argument containg the most recent group of barscore leaders in the `data.scoreLeaders` array.

As barscores increase we need to continuously re-calibrate the x-scale that calculates the width for each bar so they will not 'grow' off the screen. This is done by resetting the [d3.scale.domain](https://github.com/mbostock/d3/wiki/Quantitative-Scales#linear_domain) using [d3.extent](https://github.com/mbostock/d3/wiki/Arrays#d3_extent) to figure out the min and max barscore values.

```javascript 

function drawBarScoreLeaders(data){

		var bsLeaders = data.scoreLeaders,
			bars,
			bar;
		
		// calibrate the the bar width x-scale with newly loaded min/max barscore values
		bsx.domain(d3.extent(bsLeaders, function(d) { return d.BAR_SCORE; })).nice();
		
```

In order to nicely update players who are allready displayed on the screen, instead of simply re-drawing the entire bar chart everytime we refresh the data, we need to bind existing DOM elements to the newly loaded data.


```javascript
		// bind existing DOM elements for leaders to new data
		bars = bsViz.selectAll(".bs-leader")
				.data(bsLeaders, function(d) { return d.USER_ID; });
```


###Constructing New Barscore Leaders

To display newly arrived leaders our first step is to create a svg group that will be our container for all DOM elements. This is convenient as we can assign the css class `.bs-leader` for applying all css formatting.

```javascript

		// CREATE nodes for each leader
		bar = bars.enter()
				.append("g")
				.attr("class","bs-leader");

```

Now we append a svg rectangle for our barscore 'bar'.

```javascript

		// the bar
		bar.append("rect")
			.attr("width", function(d) {
				return bsx(d.BAR_SCORE) + minBarWidth;
			})
			.attr("height", barHeight)
			.attr("x", 0)
			.attr("y", -barHeight/2);

```

In order to turn the 'bar' into a TRON look-a-like beam we need some more svg and css magic.

```html

<style>

	.barscore rect {
		fill: url(#barGradient);
	}

</style>

...

<div style="height: 0;">
	<svg>
	  <defs>

	    	...

	    	<linearGradient id="barGradient" x1="0%" y1="0%" x2="0%" y2="100%">
	      		<stop offset="0%" style="stop-color:rgb(98,143,153);stop-opacity:1" />
	      		<stop offset="75%" style="stop-color:rgb(199,252,252);stop-opacity:1" />
	      		<stop offset="100%" style="stop-color:rgb(54,98,97);stop-opacity:1" />
	    	</linearGradient>

	  </defs>
	</svg>
</div>

```

> Note that when you look really closely at our design, it showed a rounded edge for each bar. This was not implemeneted... no other excuse that we simply ran out of time.

Now that the 'bar' is constructed lets append svg text elements for a screen name and a barscore label and 2 svg circles creating the frame for the player picture.

```javascript

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

		// add photo outer frame
		bar.append("circle")
			.attr("class", "outerCircle")
			.attr("r", radius + padding)
			.attr("cx", 0)
			.attr("cy", 0);
		
		// add photo frame
		bar.append("circle")
			.attr("class", "picFrame")
			.attr("r", radius + picFrameWidth) 
			.attr("cx", 0)
			.attr("cy", 0);
			
```

And finally we add the player picture which is a svg image.

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

In order to get the circular photos we need to create a clipping path.

```javascript
		// append the clipping path with the correct dimensions
		svg = d3.select("#clipping").append("circle")
			.attr("cx", 0)
			.attr("cy", 0)
			.attr("r", radius);
```

Which we can then reference from our `.photo` css definition.

```css
<style>
	...
	
	#leaderboard .photo {
		clip-path: url(#clipping);
	}
	
	...
</style>
```




###Remove Unnecessary Players 

We only want to display the top players, therefore those who have been outperformed by the current leaders need to go. For this we use  [selection.exit()](https://github.com/mbostock/d3/wiki/Selections#exit) and simply remove the player 'bars' that are not needed.

```javascript

		// DELETE un-needed leader nodes
		bars.exit().remove();

```

###Update Existing Leaders

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





## 2.The Beacon Leaderboard

The Beacon Leaderboard shows the players who were most successful at collecting beacons and highlights the ones they have claimed.

Looking at the source you will find `drawBeaconLeaders()` which expects a `data` argument containg the most recent group of barscore leaders in the `data.claimLeaders` array.

Our first task is to again bind DOM elements previously created for existing players with the newly loaded data. 

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

###Constructing New Beacon Claim Leaders

Again, we start with creating a svg group that will be our container for all the DOM elements and assign a css class. This time we name it `.beacon-leader`.

```javascript 
	
	// CREATE a node for each new leader
	row = rows.enter()
			.append("g")
			.attr("class","beacon-leader");

```

Then we append a svg text element for the screen name and 2 svg circles creating the frame for the player picture, as well as a svg image for the picture. As described above by applying the css class `.photo` we get a nicely rounded image.


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
		
	...
	
	// add the photo
	row.append("image")
		.attr("class", "photo")
		.attr("xlink:href", function(d) { return imgUrl(d); })
		.attr("x", -radius)
      	.attr("y", -radius - radius/2 )
		.attr("width", radius*2)
  		.attr("height", radius*1.5*2);


```

I mentioned earlier that we chose to use specific symbols inspired by TRON to represent each beacon hidden in the club. Beacons that have been claimed by a user would be highlighted, unclaimed beacons grayed out.

To accomplish this we created svg files for each beacon and used them as the src for a svg image element. In our case we had 2 svg files for each symbol, one highlighted, the other one grayed out. 

> Note: There is most likely a more elegant approach to this. Something that would use a svg path and then use css to change the color of a symbol, but given our very aggressive timeframe we did not have time for 'elegance'. And this worked just fine.

The game allowed the collection of a total of 6 beacons. The following only shows the code for one of them... the ARJIAN beacon.

During construction, all we have to do is create a svg image that does not have a source just yet. For the purpose of conveniently selecting and updating the image we apply the css class `.beacon-1`.

```javascript

	// add a ARJIAN symbol
	row.append("image")
		.attr("class", "beacon-1")
		.attr("width", symbolSize)
	  	.attr("height", symbolSize)
		.attr("transform", "translate(" +  symbolStartPos  +"," + ( -symbolSize/2 ) + ")");


```

###Remove Un-needed Players 

Just like for the Barscore Leaderboard, we only want to display the top players...

```javascript

	// DELETE un-needed leader nodes
	rows.exit().remove();

```

###Update Extisting Beacon Claim Leaders

And again for players who are already displayed from a previous data load we need to update their vertical position, their screen name and photo.

```javascript

	// UPDATE existing leader nodes
	rows.transition().duration(animDuration).ease("exp-out")
		.call(positionLeader);
	
	// UPDATE name	
	rows.select(".name")
		.text(function(d, i) {
			return (i+1) + '.' + d.NAME;
		});
		
	// UPDATE photo
	rows.select(".photo")
	    .attr("xlink:href", function(d) { return imgUrl(d); });
	    

```

And at last, to highlight the beacons they have claimed we can simply select the symbols and set the correct src for the svg image element depending on the players beacon collection value `d.BEACON_1`. 

```javascript

	// UPDATE ARJIAN symbol
	rows.select(".beacon-1")
		.attr("xlink:href", function(d) { return d.BEACON_1 === 1 ? 'svg/arjian_green.svg' : 'svg/arjian.svg'; });


```

That is folks, I hope this write up is somewhat helpful.





