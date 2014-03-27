### The 'People At The Bar' Visualization

We figured the best way to visualize micro-location detection was to simulate typical 'people at the bar' behaviour, to display their mugshot and screen name and attempt to engage them with game related personalized messages. All we needed was an interesting way to mimick people bumping into each other, trying to push through the crowd to order a drink and having a good time.

And here is what we came up with:

![alt text](/img/people-at-bar.jpg "At The Bar Visualization")

People at the bar are visualized as bubbles floating on the screen. New arrivals come bouncing onto the screen from above causing movement and eventually settling down until somebody else arrives. They are greeted with a personalized message displayed as a call-out offering game playing tips or congratulating them on their performance. As more people crowd the space and bump into each other more movement happens… you know, kind of typical bar behaviour.

### Fixed Screen Size and Other Congigurations

To keep things as simple I decided to implement this with a fixed screen size. Our A/V guys provided screens with 1900x1060 resolution so that is what I used. Looking at the source code you will notice several variables being set at the top, `vizWidth` and `vizHeight` are used to set the dimensions of the visualization.

Looking at `margin`, `width` and `height`, note that I am using Mike Bostock's [margin convention](http://bl.ocks.org/mbostock/3019563).

Others worth mentioning are `radius` which sets the people-bubble radius and `radiusBig`which is the bubble radius when people first arrive on the sreen. After a little while the bubble animates to get reduced the `radius`.

The rest of these should be quite self explanatory after reading the comments.

```javascript
// CONFIGS
var svg, // the svg canvas
	barViz, // the svg group containing the bar visualization
	barCount = 10, // the max amount of people on the screen 
	refreshTime = 7000, // milliseconds between data refresh requests 
	vizWidth = 1900, //1920, // total available screen width
	vizHeight = 1060, //1080, // total available screen height
	margin = {top: 30, right: 30, bottom: 30, left: 30}, // margin object
	width =  vizWidth - margin.left - margin.right, // calculated width of visualization
	height = vizHeight - margin.top - margin.bottom, // calculated height of visualization
	padding = 13, // padding between people-bubbles
	animDuration = 2000, // the time it takes to animate a change
	radius = 100, // bubble radius
	radiusBig = 200, // new arrival bubble radius
	textPadding = 22, // the padding between the photo frame and the screen name label
	picFrameWidth = 2, // the stroke-width of the circle border around the photo
	lastLoadedPeople = [], // an array containing the data for most recently loaded people
	peopleOnScreen = [], // an array containing the data for people currently on the screen
	force; // the force layout
	
```

### New Arrival Messages

Our design specified that we wanted to greet people arriving to the bar with game related personalized messages. The messages should be intelligent enough so they would match a persons current game status and encourage them to improve it.

To keep this somewhat simple I set up several arrays containing message objects, one for each of the following scenarious:

- Newcomers
- Players without a photo
- Players with various bar score levels
- Players with various amounts of beacons collected

To allow for various message formats, each message object contained a greeting, a start and and end-text snippet. Depending on the message type a barscore of beacon count would be inserted between start and end...


```javascript
// CALLOUTS for Newcomers
// does NOT use score
var newcomerSnippets = [
	{
		greeting: 'Hi',
		start: 'Get a drink',
		end: 'to boost your bar score!'
	}, {
		greeting: 'Yo',
		start: 'Find a beacon to play the game!',
		end: ''
	},
	...
			
```

The logic for picking the right message is implemented using a very simplistic decission tree. The passed data object contains the information needed to decide which message to return. Nothing sophisticated, but it seemed to work quite well faking a somewhat intelligent display.

```javascript
function getCalloutTxt(d){
	
	var barscore = d.BAR_SCORE,
		picture = d.IMG_REF,
		claims = d.TOTAL_CLAIMS,
		name = d.NAME,
		type = function(){
			// return a random type
			var types = ['barscore','claims'];
			return types[Math.floor(Math.random() * types.length)];
		},
		score,
		snippet,
		snippets;
	
	// newcomers
	if(barscore === 1){				
		snippets = newcomerSnippets;
		score = '';
	}
	
	// no Picture
	else if(picture === null){				
		snippets = noPhotoSnippets;
		score = '';
	}
	...
}
```


### Constructing a Person

So by now you already know that each person is displayed as a floating bubble on the screen. The composition of these bubbles are best illustrated with a sketch.

![alt text](/img/people-bubble.svg "People Bubble Sktech")

The code to implement this is pretty standard D3 stuff. If you are not familiar with D3 first take a look at:

[D3 selection](https://github.com/mbostock/d3/wiki/Selections)

[D3 selection.data](https://github.com/mbostock/d3/wiki/Selections#data)

So assuming you are ow familiar with selecting extisting DOM elements and binding them to data, here is how a person-bubble is constructed.

> Note that the following code does not only implement the creation of various DOM elements but also takes care of the animation that happens when a person is created on the screen. Initially the person bubble is larger and is displayed with a callout. After a little while the bubble animates to a smaller size and the callout blends out.

So lets get started. First create a svg group that will be our container for all the elements. This is convenient as we can assign a css class, 'person' in this case, for applying styles conveniently.

```javascript
// create a node for the person
person = barViz.selectAll(".person")
    .data(peopleOnScreen, function(d) { return d.USER_ID; })
	  .enter().append("g")
	  	.attr("class", "person")
	  	.attr("userId", function(d){ return d.USER_ID; });
```

Then append a svg circle which will be our photo frame.

```javascript		    
// add a bubble
person.append("circle")
    	.attr("class", "photo-frame")
    	.attr("r", radiusBig + picFrameWidth)
    	.transition().delay(timeShowingCallout).duration(animDuration).ease("exp-in")
			.attr("r", radius + picFrameWidth);
```

Now add the photo. To clip the image in so it is displayed in a nice circle we need to append a clip-path first and then the image.

```javascript
// add a clipping path for the photo
person.append("clipPath")
		.attr("id", function(d) { return "clip-path-" + d.USER_ID; })
		.append("circle")
			.attr("cx", 0)
			.attr("cy", 0)
    		.attr("r", radiusBig)
    		.transition().delay(timeShowingCallout).duration(animDuration).ease("exp-in")
				.attr("r", radius);
			
// add the photo
person.append("image")
	.attr("class", "photo")
	.style("clip-path", function(d) { return "url(#clip-path-" + d.USER_ID +")"; })
	.attr("xlink:href", function(d) { return imgUrl(d); })
	.attr("x", -radiusBig)
      .attr("y", -radiusBig - radiusBig/2 )
	.attr("width", radiusBig*2)
  	.attr("height", radiusBig*1.5*2)
	.transition().delay(timeShowingCallout).duration(animDuration).ease("exp-in")
		.attr("x", -radius)
      	.attr("y", -radius - radius/2 )
		.attr("width", radius*2)
  		.attr("height", radius*1.5*2);

```	  				

And last the callout containing our personalized message

```javascript
  // add a callout
person.append("image")
	.attr("class", "callout")
	.attr("width", calloutW)
  	.attr("height", calloutH)
  	.attr("xlink:href", 'svg/callout.svg')
  	.style("opacity", 0)
	.attr("transform", "translate(" +  (radiusBig-80)  +"," + ( -radiusBig*2 ) + ")")
	.transition().duration(animDuration).ease("exp-out")
		.style("opacity", 1)
	.transition().delay(timeShowingCallout).duration(animDuration).ease("exp-out")
		.style("opacity", 0)
		.remove();
			
// add the callout txt
calloutTxt = person.append("foreignObject")
		.attr("class", "callout-txt")
        .attr('x', radiusBig-80)
        .attr('y', - radiusBig*2)
        .attr("width", calloutW)
        .attr("height", calloutH)
      .append("xhtml:p")
        .html(function(d) { return getCalloutTxt(d); } );

calloutTxt.transition().delay(timeShowingCallout).duration(animDuration).ease("exp-out")
		.style("opacity", 0)
		.remove();				

```

And at last we need a name label that is displayed after the bubble has been resized and the callout disappears.

```javascript
  // add the partner name
person.append("text")
	.attr("class", "name")
	.attr("y", radius + textPadding)
	.style("opacity", 0)
	.text(function(d) { return d.NAME; })
	.transition().delay(timeShowingCallout+1000).duration(animDuration).ease("exp-out")
		.style("opacity", 1);  					
  				
```


### The Animated Layout

To auto-position and animate the people I am using [d3.layout.force](https://github.com/mbostock/d3/wiki/Force-Layout#force) which constructs a force-directed layout and does all of the complicated math for us to simulate simulate the physics of bubbles floating and bumping off each other and the sides.

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

After playing around with various configurations I found that using `gravity` and `charge` are sufficient to accomplish the desired effect. 

`Gravity` simulates just that and adds continuos motion giving us the 'floating' we are looking for.
`Charge` gives each bubble somthing like a negative charge so bubbles bounce off each other.

Now all we have to do is implement a callback for each 'animation' tick. The force layout already calculates x/y positions for a bubble, however we need to make sure bubbles are not bouncing off the screen. This can be implemented with a layout helper `contain()`.

Here is how it all comes together.

```javascript
/**
 * called on every tick of the 'force' layout while it is running
 */
function onForceTick(e){
	
	barViz.selectAll(".person")
      	.each(contain())
      	.attr("transform", function(d) { return "translate(" + d.x + "," + d.y +")"; } );
      	
}


/**
 * contain person within the screen area
 */
function contain(){
	
	return function(d){	
		
		var limit = radius + textPadding;
		
		if( d.x > width - limit ) {
			d.x = width - limit ;
		}
		else if( d.x < limit ){
			d.x = limit;
		}
		
		if( d.y > height - limit ) {
			d.y = height - limit;
		}
		else if( d.y < limit ){
			d.y = limit;
		}			
		
	};
}
```



### Getting 'Live' Updates from the Server

- Describe how to poll the server
- Describe how to establish the delta


### Newcomers, Updates and Departures

- Why we can simply apply newly loaded data 




