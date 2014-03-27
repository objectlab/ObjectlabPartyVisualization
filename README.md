# Party App Visualizations

For our party game app we implemented two visualizations to give players a way to show off their game play performance and make party guest aware of the game and hopefully entice them to participate.

### D3.js

When it comes to HTML based visualizations the choice is easy. If you have not heard about D3.js, it is a JavaScript library for manipulating documents based on data using HTML, SVG and CSS. For more information check out [D3.js](http://www.d3js.org)


### The 'At The Bar' Visualization

We figured the best way to visualize micro-location detection was to simulate typical 'people at the bar' behaviour, to display their mugshot and screen name and attempt to engage them with game related personalized messages. All we needed was an interesting way to mimick people bumping into each other, trying to push through the crowd to order a drink and having a good time.

And here is what we came up with:

![alt text](/img/people-at-bar.jpg "At The Bar Visualization")

People at the bar are visualized as bubbles floating on the screen. New arrivals would come bouncing onto the screen causing movement eventually settling down until somebody else arrives. They would be greeted with a personalized message displayed as a call-out offering game playing tips or congratulating them on their performance. As more people crowd the space and bump into each other more movement happens… you know, typical bar behaviour.

[d3.layout.force](https://github.com/mbostock/d3/wiki/Force-Layout#force) constructs a force-directed layout implementing concepts of attraction/repulsion, friction, proximity, charge and gravity along with others. These are all the ingredients needed to accomplish the animation we are looking for.

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


### The 'Leaderboard' Visualization






