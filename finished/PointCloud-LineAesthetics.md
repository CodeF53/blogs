# Line Aesthetics of PointClouds
If you somehow got here without coming from my dev log on PointClouds, I recommend you read that first. You will be able to find it [here, on my dev blog](https://dev.to/f53)

This is a extension on that for people who want more detail on a bit I glossed over in the main thing.

## The Plan

Here are the ideas I went in with when changing the line render code:
- further away = more transparent
- further away = less thick
- very far away = dont draw at all.

## The method in general

Here is how I did that, dont worry I will explain this line by line.

```js
function drawLine(point0, point1) {
    // literally all the logic set above is based on the length of the line, so we calculate that.
    distance = dist(point0, point1);

    // WHOA, what??!?!??!?! lets slow down and explain this one in detail.
    transparency = ((1-(distance / screenMaxLength))-0.75)*5;
    if (transparency > 0) {
        ctx.lineWidth = transparency*2;
        if (transparency>=1){
            transparency=1;
        }
        ctx.strokeStyle = `#FFFFFF${(parseInt(transparency*255)).toString(16)}`;

        ctx.beginPath();
        ctx.moveTo(point0.x, point0.y);
        ctx.lineTo(point1.x, point1.y);
        ctx.stroke();
    }
}
```

## Calculating transparency.

Here is the logic behind that transparency value: 

(as you read, imagine that 0 is no line and 1 is solid white line)
```js
// this gives us a good value for how long the line is relative to the size of the user's screen.
// basically, this would be 0 if the 2 points were on top of eachother, and 1 if the points were on opposite corners of the screen
distance / screenMaxLength

// inverting that gives us 1 if they are close, and 0 if they are far, kinda what we want
1-(distance / screenMaxLength)

// to bias it towards shorter lines, we first reduce it
(1-(distance / screenMaxLength))-0.75
// new interval is (-0.75, 0.25) 
// having a negative tranparency is problematic, so this is where our if statement comes in 
// by adding a check for if (transparency > 0){},
// we change our interval to (0, 0.25)
// this means that we have effectively made it so only lines that are shorter than 1/4 of screenMaxLength are drawn.

// but now we are only drawing lines at 0.25% transparency
// to fix that, we could multiply by 4, setting our interval to (0, 1)
// but instead, we actually multply by 5, giving us an interval of (0, 1.25)
((1-(distance / screenMaxLength))-0.75)*5
// this makes a bit more lines completely solid aswell as letting us do a little trick with the width of lines that are really close together
```

## A bit more of the method explained in general

Whoa, now that that one line is explained, lets look at the rest of the code.

```js
// calculate transparency as a number between -0.75 and 1.25
transparency = ((1-(distance / screenMaxLength))-0.75)*5;

// dont draw lines longer than 25% of the screenMaxLength
// narrows transparency interval to (0, 1.25)
if (transparency > 0) {
    // lines get thicker the closer points are to each other
    // ex) really far, transparency = 0.1, the line will only be 0.2px thick
    // ex) really close, transparency = 1.25, the line will be 2.5px thick
    ctx.lineWidth = transparency*2;
    // this makes a subtle but dramatic improvevment to how our final result looks

    // we need to narrow transparency interval to (0, 1) for the next bit
    // so we do that here.
    if (transparency>=1){
        transparency=1;
    }

    // another really hard to explain thing, lets break out again.
    ctx.strokeStyle = `#FFFFFF${(parseInt(transparency*255)).toString(16)}`;

    // we already went over this bit, so it should still make sense to you
    ctx.beginPath();
    ctx.moveTo(point0.x, point0.y);
    ctx.lineTo(point1.x, point1.y);
    ctx.stroke();
}
```

## Making our transparency value actually usefull.

```js
ctx.strokeStyle = `#FFFFFF${(parseInt(transparency*255)).toString(16)}`;
```
This basically just converts our transparency value into a color that canvas can understand, but lets go into it.

```js
// this is basically just setting the pen color to white.
// in hexadecimal FF = 255, so this is saying 
// 255 red, 255 green, 255 blue
ctx.strokeStyle = `#FFFFFF`

// the next 2 digits we add on to this will be the transparency
// (opacity really, where 255 is completely solid, and 0 is completely see-through)
// for example, if we did
ctx.strokeStyle = `#FFFFFFFF`
// the line would have 100% opaque and would render completely solid, and you couldnt see anything through it.

// vise versa, giving it 00 for opacity would give us a line that is completely see through (invisible)
ctx.strokeStyle = `#FFFFFF00`

// so how do we convert our transparency value, that is currently (0, 1) to this, which should be (0, 255) formatted in hexadecimal?

// first we can change our interval to match that of our target
// this is simple as multiplying by 255
transparency*255

// at this point, our number probably looks something like this:
// 227.3854753 or 22.1874386
// we dont want all this preciciosn, so we parse it down to an int.
parseInt(transparency*255)

// now we have a integer number, going from 0-255
// how do we get it to hexadecimal?
// luckily, javascript has an built in way to convert ints in base 10 to ints in any base!
// integer.toString(base)
(parseInt(transparency*255)).toString(16)

// now that we have our opacity in 2 digits of hexadecimal
// its as simple as adding that to the end of our hex color code.
`#FFFFFF${(parseInt(transparency*255)).toString(16)}`
```

### Line Aesthetics - Summary

With that explained, lets take a look back at the method to sum up everything.

```js
// distance is a usefull number because all our logic centers around it.
distance = dist(point0, point1); 

// normalize our distance to a number between 0 and 1.25
transparency = ((1-(distance / screenMaxLength))-0.75)*5;
if (transparency > 0) {
    // lines get thicker the closer they are.
    ctx.lineWidth = transparency*2;

    // but make sure transparency never goes above 100%
    if (transparency>=1){
        transparency=1;
    }
    // convert transparency from 0-1 to 0-255 in hexadecimal,
    // inorder to use it in a hex color code for our line
    ctx.strokeStyle = `#FFFFFF${(parseInt(transparency*255)).toString(16)}`;

    // draw the line.
    ctx.beginPath();
    ctx.moveTo(point0.x, point0.y);
    ctx.lineTo(point1.x, point1.y);
    ctx.stroke();
}
```

With that we have a very nice looking pointcloud.
![image of a very nice looking pointcloud](https://i.imgur.com/iyVA4FR.png)

(you can go back to the original article now, kthxbai)