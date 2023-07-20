## ToolTips

I wanted some tooltips for my website.

Problem was none of the online tutorials for tooltips actually behave like a tooltip, with the text following your mouse cursor.

I spent 3 hours figuring out how to make theme work the way I wanted.

### HTML

Add a `tooltip` attribute to all the elements you want to have a tooltip.

The value of this attribute should be what you want the tooltip's text to be. You can use HTML here

```html
<!DOCTYPE html>
<html>
    <head>
        <link rel="stylesheet" href="style.css" />
        <script src="index.js"></script>
    </head>
    <body>
        <a tooltip="Example Tooltip Text A" href="www.example.com">A</a>
        <a tooltip="Different<br>Tooltip<br>Text" href="www.otherExample.com">B</a>
        <a tooltip="Really Cool Site" href="f53.dev">C</a>
    </body>
</html>
```

You are also going to need to link a javascript file somewhere

### CSS

We need a very small amount of CSS to make our tooltip render right.

```css
body {
    /* Required Stuff */
    position: relative;

    /* Styling that fits tooltip example styling */
    background: #1c1c1c;
}

#tooltip {
    /* Required Stuff */
    position: absolute;
    pointer-events: none;
    z-index: 100; /* anything higher than the highest z index you use */
    visibility: hidden;

    /* Example of how to style your tooltip */
    background: rgba(255,255,255,0.1);
    backdrop-filter: blur(5px);
    color: white;
    font-size: 1em;
    padding: 0.5em;
    border-radius: 1em;
}
```

### JS

The meat of this is in the javascript so I will be breaking it down slower here.

If you put the script in the header like I recommended then all your code needs to be wrapped in this

```js
addEventListener('DOMContentLoaded', () => {
    // Everything
})
```

We need to make our tooltip display and append it to the body

```js
// Make our tooltip
let tooltip = document.createElement("span")
tooltip.id = "tooltip"
document.body.appendChild(tooltip)
```

We add event listeners for all our elements that have a tooltip, calling a function that will handle everything

```js
// add event listeners to elements that have a tooltip
document.querySelectorAll("[tooltip]").forEach(element => {
    element.addEventListener("mouseenter", updateTooltip)
    element.addEventListener("mouseleave", updateTooltip)
    element.addEventListener("mousemove", updateTooltip)
})
```

Then we just need to make the function those event listeners were calling.

```js
function updateTooltip(mouseEvent) {
    // Move tooltip to our current cursor position
    tooltip.style.top = mouseEvent.pageY+"px"
    tooltip.style.left = mouseEvent.pageX+"px"

    switch(mouseEvent.type) {
    case "mouseenter":
        // update text and show tooltip when we hover
        tooltip.innerHTML = this.getAttribute("tooltip")
        tooltip.style.visibility = "visible"
        break;
    case "mouseleave":
        // hide the tooltip when we are no longer above a tooltip element
        tooltip.style.visibility = "hidden"
        break;
    }
}
```

## Conclusion and Demo

That is it!

You can play around with the code for this [here](https://codepen.io/codef53/pen/QWmzpVw)
