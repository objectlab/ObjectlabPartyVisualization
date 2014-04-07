Leaderboard Visualization
=============

The Leaderboard visualization needed to show the current leaders in both game play objectives.

First, a leaderboard for the highest barscore. We figured the more people frequented the bar the more drinks they had to be consuming.  Therefore the complicated mathematical equation of:

> high barscore = high level of drunkenness

The second game objective would be to claim all of the beacons hidden in the club. The party had somewhat of a TRON related theme, so to visualize each beacon we borrowed some nerdtastic TRON symbols.

For both leaderboards wanted to incorporate the players’ pictures and their screen names and add a little animation into the mix.

And here is what the end result looks like:

![alt text](/img/visualization-banner.jpg "Leaderboard Visualization")

The Barscore Leaderboard is basically a bar chart with screen name labels on the left and barscores displayed on the right. To tie this to the ‘At The Bar’ Visualization we used bubbles for each player to display their selfie. We would show the leading players on the top, new leaderboard arrivals would animate onto the screen from above pushing less successful players off the screen, position changes would animate respectively.

### Fixed Screen Size and Other Configurations

Again, similar to the ['People At The Bar' Visualization](/bar/) I decided to implement this with a fixed screen size of 1900x1060 resolutionusing Mike Bostock's [margin convention](http://bl.ocks.org/mbostock/3019563).

Looking at the source code you will notice several variables being set at the top. They should be quite self explanatory after reading the comments.

```javascript
		
		// SHARED vars
		var 	svg, // the svg canvas
			leaderCount = 8, // number of leaders shown
			refreshTime = 7000, // milliseconds between data refresh requests
			vizWidth = 1900, // total screen width
			vizHeight = 1060, // total screen height
			margin = {top: 80, right: 90, bottom: 60, left: 330}, // the margins of each leaderboard
			padding = 13, 
			animDuration = 2000, // the time it takes to animate a change
			width =  vizWidth/2 - margin.left - margin.right, 
			height = vizHeight - margin.top - margin.bottom, // individual leaderboard height
			radius = ( height - padding*leaderCount ) / ( leaderCount*2 ),
			picFrameWidth = 2;
		
		// BEACON vars	
		var 	beaconViz;
			
		// BARSCORE vars
		var 	bsViz,
			barHeight = 30,
			minBarWidth = radius + padding + 10,
			bsx = d3.scale.linear().range([0, width-minBarWidth]);
```


