# Intro
Often sites will blur content that they want you to pay or sign up to access.

If they do it wrong, its super easy to bypass.

## Example
https://www.jobscan.co/ is a site that helps optimize your resume to be read by a bot. It requires you to sign up to view the advice it gives:

![](https://i.imgur.com/nZ8qTWh.png)

By opening devtools, and pasting the following javascript, you can just unblur everything:

```js
[...document.querySelectorAll(".obfuscationWrapper")].forEach(node => node.className="")
```

![](https://i.imgur.com/POPBNXs.png)

It now shows the sign up for free box all the time too, but that is ok

## How to see if a site is susceptible to this:
Open your dev tools, go to Inspector, and use the element picker to find the blurred content:
![](https://i.imgur.com/8EnuIw2.png)
![](https://i.imgur.com/DR1REV1.png)

If the blurred content is just wrapped by a div with a class implying blur/obfuscation/censorship, its susceptible.

If it's an image, its not.

## How does the code to reveal the content work?
We start by searching the document for all the nodes that have the class that blurs their content:
```js
document.querySelectorAll(".obfuscationWrapper")
```

`querySelectorAll` outputs a weird thing you cant iterate through, so we have to spread it out into an array:
```js
[...document.querySelectorAll(".obfuscationWrapper")]
```

then, we use `forEach` to run code to remove the classes from each node
```js
[...document.querySelectorAll(".obfuscationWrapper")].forEach(node => node.className="")
```

All in all, this code finds nodes with the `obfuscationWrapper`, and removes their classes.

Turning:
```html
<div class="obfuscationWrapper">
```

Into:
```html
<div>
```

This makes css that blurs these nodes not work anymore, because they cant target a class that doesn't exist:
```css
.obfuscationWrapper::after {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  backdrop-filter: blur(8px);
  content: "";
}
```