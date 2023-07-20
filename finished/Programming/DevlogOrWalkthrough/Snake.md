## Intro

I was bored and wanted to make a thing. How about snake?

Snake is a simple game that works on a grid.

- once you start, the snake is always moving
- when the snake runs into itself it dies.
- eating apples expands the snake
  - apples are placed randomly

The goal of the game is to eat as many apples as possible without dying.

## Game Controls

We gotta have a way for a user to change the direction our snake is facing, so they can play the game.

There are two "obvious" control layouts for this:

- WASD: Your standard controls for every pc game
  - W "up/forward"
  - A "left"
  - S "down/backward"
  - D "right"
- Arrow keys: its pretty intuitive to have these keys mapped to directions
  - Up "up"
  - Left "left"
  - and so on...

I didn't want to force people to choose one layout, so I just decided to implement both.

Setting up these controls is brutally simple, just adding an event listener to keypresses, with a bit of logic from a switch.

```js
let Facing = ""
// Game Controls
document.body.addEventListener("keydown", (e) => {
    switch(e.code) {
        case "KeyW":
        case "ArrowUp":
            Facing = "up"
            break;
        
        case "KeyA":
        case "ArrowLeft":
            Facing = "left"
            break;
        
        case "KeyD":    
        case "ArrowRight":
            Facing = "right"
            break;
        
        case "KeyS":
        case "ArrowDown":
            Facing = "down"
            break;
    }
}); 
```

## A clever way to wait until a user's first input

The game will run in "ticks", using a similar interval thing to the pointcloud code. I chose 50ms randomly.

```js
setInterval(()=>{gameTick();gameRender()}, 50);
```

We don't want our game to instantly start without the user being ready. Because we initialize our `Facing` variable as an empty string, we can just check if the string is empty or not.

```js
function gameTick() {
    if (Facing != "") {
        // do game logic
    }
}
```

## Snake Storage

You could rack your brain for hundreds of ways to store the positions of a snake, I went with this.

We will have this array of objects I will call snakeparts for simplicity.

```js
snake = [{x:13,  y:15}, {x:14,  y:15}, {x:15,  y:15}]
```

The very last snakepart in the array is the "head", every gametick we will add a snakepart after this head, offsetting it from the prior head given the direction the snake is facing.

```js
let oldHead = snake[snake.length-1]
switch (Facing) {
    case "up":
        snake.push({x:mod(oldHead.x,  GameSize), y:mod(oldHead.y-1,GameSize)})
        break;
    case "left":
        snake.push({x:mod(oldHead.x-1,GameSize), y:mod(oldHead.y,  GameSize)})
        break;
    case "right":
        snake.push({x:mod(oldHead.x+1,GameSize), y:mod(oldHead.y,  GameSize)})
        break;
    case "down":
        snake.push({x:mod(oldHead.x,  GameSize), y:mod(oldHead.y+1,GameSize)})
        break;
}
```

We use mod here for the exact same reason we used mod to let the points wrap around the canvas in the pointcloud.

```js
const mod = (num, val) => ((num % val) + val) % val;
```

The very first snakepart is the "tail", every gametick we will remove it from the array if the snake hasn't eaten.

```js
if (__Snake_Ate_Apple__) {
    // put apple in new position
} else {
    // remove the snake's tail
    snake.splice(0,1);
}
```

## Apple Logic

```js
let apple = {x:rand(GameSize), y:rand(GameSize)}
```

Our apple will be stored in a single object, identical in formatting to a single snakepart.

Our random here is different from the one we used in PointCloud because we are working on a small grid, so want integers, and we will also always have a min of 0

```js
const rand = (max) => parseInt(Math.random() * max); // random int between 0 and max
```

Checking if our snake has "eaten" the apple is super simple, we just look to see if the head is in the same position as it.

```js
let headPos = snake[snake.length-1]
// check if successfully ate apple 
if (headPos.x == apple.x && headPos.y == apple.y) {
    // new apple
} else {
    // remove the snake's tail
    snake.splice(0,1);
}
```

After we have eaten the apple, we need a new apple. This is a bit complex.

While we could just say a new random location, like this:

```js
if (headPos.x == apple.x && headPos.y == apple.y) {
    // new apple
    apple = {x:rand(GameSize), y:rand(GameSize)}
} else {
```

This new location could be covered by a part of the snake's body, leading to confusion.

To fix this, we use the following logic

- Make a new apple at a random location
- Check if any part of the snake is overlapping
- If it's overlapping, try a new location

```js
let invalidApplePos = true;
while (invalidApplePos) {
    invalidApplePos = false
    apple = {x:rand(GameSize), y:rand(GameSize)}
    // make sure snake isn't overlapping apple
    for (let i = 0; i < snake.length; i++) {
        if (snake[i].x == apple.x && snake[i].y == apple.y) {
            invalidApplePos = true
        }
    }
}
```

The general "win" condition for snake is when you have covered the whole board.

This is great, because it stops the user from playing anymore once they have won because the while loop never terminates

## Snake Death

The last core part of the game we need to implement is a way for our snake to "die"

We can just add a simple iterator for this after our apple logic.

```js
for (let i = 0; i < snake.length-1; i++) {
    if (headPos.x == snake[i].x && headPos.y == snake[i].y) {
        // die
    }
}
```

But what happens when we die?

## Game restarts

Instead of initializing all of our game variables with an initial value, we can do this:

```js
let Facing
let snake
let apple 
function setGameVars() {
    Facing = ""
    snake = [{x:GameSize/2-1,  y:GameSize/2}]
    apple = {x:rand(GameSize), y:rand(GameSize)}
}
setGameVars()
```

With this, when we want our game to end, we can call `setGameVars()` to reset everything.

## Putting the game logic together

With everything we have gone over, we have our game!

```js
// Math Functions
const rand = (max) => parseInt(Math.random() * max); // random int between 0 and max
const mod = (num, val) => ((num % val) + val) % val; // modulus operator, what % should be.

// Game Variables
const GameSize = 30
let Facing
let snake
let apple 
function setGameVars() {
    Facing = ""
    snake = [{x:GameSize/2-1,  y:GameSize/2}]
    apple = {x:rand(GameSize), y:rand(GameSize)}
}
setGameVars()

// Game Controls
document.body.addEventListener("keydown", (e) => {
    switch(e.code) {
        case "KeyW":
        case "ArrowUp":
            if (Facing != "down")
                Facing = "up"
            break;
        
        case "KeyA":
        case "ArrowLeft":
            if (Facing != "right")
                Facing = "left"
            break;
        
        case "KeyD":    
        case "ArrowRight":
            if (Facing != "left")
                Facing = "right"
            break;
        
        case "KeyS":
        case "ArrowDown":
            if (Facing != "up")
                Facing = "down"
            break;
    }
}); 

function gameTick() {
    if (Facing != "") {
        let oldHeadPos = snake[snake.length-1]
        switch (Facing) {
            case "up":
                snake.push({x:mod(oldHeadPos.x,  GameSize), y:mod(oldHeadPos.y-1,GameSize)})
                break;
            case "left":
                snake.push({x:mod(oldHeadPos.x-1,GameSize), y:mod(oldHeadPos.y,  GameSize)})
                break;
            case "right":
                snake.push({x:mod(oldHeadPos.x+1,GameSize), y:mod(oldHeadPos.y,  GameSize)})
                break;
            case "down":
                snake.push({x:mod(oldHeadPos.x,  GameSize), y:mod(oldHeadPos.y+1,GameSize)})
                break;
        }
        let headPos = snake[snake.length-1]
        // check if successfully ate apple 
        if (headPos.x == apple.x && headPos.y == apple.y) {
            // put apple in new position
            let invalidApplePos = true;
            while (invalidApplePos) {
                invalidApplePos = false
                apple = {x:rand(GameSize), y:rand(GameSize)}
                // make sure snake isn't overlapping apple
                for (let i = 0; i < snake.length; i++) {
                    if (snake[i].x == apple.x && snake[i].y == apple.y) {
                        invalidApplePos = true
                    }
                }
            }
        } else {
            snake.splice(0,1);
        }
        // check for overlaps
        for (let i = 0; i < snake.length-1; i++) {
            if (headPos.x == snake[i].x && headPos.y == snake[i].y) {
                // die
                setGameVars()
            }
        }
    }   
}

setInterval(()=>{gameTick();gameRender()}, 50);
```

That is snake in just below 100 lines of javascript, but we are forgetting something.

## Rendering the game

I left this part last because it was the hardest part for me. I wanted to go for a Pixel Art rendering style, with a small 30x30 pixel canvas.

Rendering this way is super duper easy and self explanatory.

We initialize our canvas the same way as we did in my pointcloud code

```js
// Canvas Initialization
const canvas = document.getElementById('SnakeGame');
canvas.width = GameSize;
canvas.height = GameSize;
const ctx = canvas.getContext('2d');
```

Then in our render function, we fill the canvas black, then draw in our red apple and white snake.

```js
function gameRender() {
    ctx.fillStyle = 'black';
    ctx.fillRect(0, 0, GameSize, GameSize);

    ctx.fillStyle = "red"
    ctx.fillRect(apple.x,apple.y,1,1)

    ctx.fillStyle = "white";
    for (let i = 0; i < snake.length; i++) {
        ctx.fillRect(snake[i].x, snake[i].y, 1,1)
    }
}
```

## Making our render visible

This is all super simple and easy, but there is one huge problem, or maybe it would be better to call it a very "small" problem.

![](https://i.imgur.com/LLh1EAE.png)

Our game is only 30px by 30px, absolutely TINY.

I expected fixing this problem to be a cakewalk, with a little css, we can stretch our canvas to fit the height of the screen

```css
#SnakeGame {
    width: auto;
    height: 100%;
}
```

Or atleast I thought it would be that easy, nope, that doesn't work.

No amount of `!important` or nested div nonsense I could think of could get this game to render big.

Fixing this one thing took me nearly 4 hours of nonstop googling for documentation of resizing canvases, pixel upscaling, and Mozilla's docs on css width/height. Throughout this time, I simply refused to look up a proper tutorial on doing pixel art in javascript.

Eventually though, I cracked and searched for a prewritten solution.

I thought doing this would instantly get me to the results I needed, but no, it didn't help at all. But eventually, I got lucky and ran across [this one gist](https://gist.github.com/zachstronaut/1184900).

This was it. After hours of searching, this one post solved all of my problems, instantly.

I cut his implementation down to the bare essentials, something I could just paste at the bottom of my code.

```js
// We no longer take in an argument because we defined it in our render code.
function FitCanvas(){
    // we do our calculations for X and Y inplace
    let scale = {
        x: (window.innerWidth) / canvas.width, 
        y: (window.innerHeight) / canvas.height
    };
    // we then say we will stretch our canvas to whichever scale is less
    // this makes our square never get cut off by the edges of the window
    if (scale.x < scale.y) { 
        scale = scale.x + ', ' + scale.x;
    } else { 
        scale = scale.y + ', ' + scale.y; 
    }
    // We then add a bunch of css to our canvas
    // I will break this down outside of this function
    canvas.setAttribute('style', canvas.getAttribute("style") + ' -ms-transform: scale(' + scale + '); -webkit-transform: scale3d(' + scale + ', 1); -moz-transform: scale(' + scale + '); -o-transform: scale(' + scale + '); transform: scale(' + scale + ');');
}
// After defining this, we call it once to fit our canvas
FitCanvas()
// And add an event listener to window size changes so our canvas always fits.
window.addEventListener('resize', function () {FitCanvas();}, false);
```

It turns out the world of "universal standards" in web development is made up.
There are 5 different methods for transforming a canvas.

```css
#canvas {
    -ms-transform: scale(_calculated_scale_); 
    -webkit-transform: scale3d(_calculated_scale_, 1); 
    -moz-transform: scale(_calculated_scale_); 
    -o-transform: scale(_calculated_scale_); 
    transform: scale(_calculated_scale_);
}
```

That wasn't all the css the fullscreenify method initially contained, I just split out the constant css into my actual css file.

```css
#SnakeGame { 
    /* 5 different standards for the exact same thing */
    -ms-transform-origin: center top; 
    -webkit-transform-origin: center top;
    -moz-transform-origin: center top; 
    -o-transform-origin: center top;
    transform-origin: center top;

    /* extra css fullscreenify didn't include */
    display: block; 
    /* Makes it not upscale like Garbage */
    image-rendering: pixelated;  
    /* Centers the thing */
    margin-left: auto;
    margin-right: auto;
}
```

## Conclusion

You can play the completed game at <https://codef53.github.io/snake/>

It will eventually be at [f53.dev/snake](https://f53.dev/snake), but I need to figure out how to put it there.

I didn't plan on making a blog for this, so I wrote this in retrospect. Given that, I don't feel too happy with how this one reads, but please let me know what you think!

I didn't go anywhere near as indepth with this one because I am now at the point where my brain just speaks javascript.
