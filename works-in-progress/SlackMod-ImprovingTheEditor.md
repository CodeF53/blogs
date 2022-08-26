## Intro
In [my previous blog](https://dev.to/f53/adding-a-custom-css-menu-to-slack-1090) I made a working live custom css editor for slack inspired by discord mods. I ended that blog with the following list of limitations the mod has:

> Currently, this mod is limited in several ways:
> - CSS does not save newlines
> - CSS does not persist between restarts
> - Injecting is manual and hardcoded to the user's system
> - Selecting custom css in preferences then selecting a different category leads to an issue
> - No syntax highlighting

This blog will focus on addressing the following of the prior limitations:
- CSS does not save newlines
- CSS does not persist between restarts
- No syntax highlighting

## How do other mods do it?
I had an idea in my dream last night that looking at how Discord mods added custom CSS screens would be pretty helpful.

Instead of spending ages problem solving with trial and error to make our own editor, we can base our work on people who have already solved the same issue.

### Reviewing GooseMod's CustomCSS Code

I decided to dig through the [code for GooseMod's css editor](https://github.com/GooseMod-Modules/CustomCSS/blob/main/index.js) for inspiration and ideas.

In the first few lines we can start drawing similarities between our code and theirs.

`main/index.js lines 8-12`
```js
const updateCSS = (c) => {
  styleEl.innerHTML = '';
  
  styleEl.appendChild(document.createTextNode(c));
};
```

This is equivalent to our `updateCurrentCSS` method
```js
const updateCustomCSS = newCSS => { 
    document.querySelector("#SlackMod-Custom-CSS").innerText = newCSS; 
}
```

Looking at what they do differently, I wondered if fixing the problem where CSS was not saving newlines was as simple as changing our method to use appended text nodes instead of `innerText`.

To test this idea, I revised my methods to this
```js
// method to quickly change css
const updateCustomCSS = newCSS => { 
    document.querySelector("#SlackMod-Custom-CSS").innerHTML = "" 
    document.querySelector("#SlackMod-Custom-CSS").appendChild(document.createTextNode(newCSS); 
}
```
I also changed my get method to use innerHTML instead of innerText
```js
// method to quickly get inner css
const getCustomCSS = () => { 
    return document.querySelector("#SlackMod-Custom-CSS").innerHTML
}
```

Shockingly, this worked! My text editor actually kept it's newlines! I would've never thought to try that.

Looking at the remaining code, it looks like they use a library called "ace" to create a nice looking text editor

First they import the libraries they will use

`main/index.js lines 22-27`
```js
// Setup ace
eval(await (await fetch(`https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js`)).text()); // Load Ace main

eval(await (await fetch(`https://ajaxorg.github.io/ace-builds/src-min-noconflict/theme-monokai.js`)).text()); // Load Monokai theme

eval(await (await fetch(`https://ajaxorg.github.io/ace-builds/src-min-noconflict/mode-css.js`)).text()); // Load CSS lang
```

Then they call a function from the goosemod API to add a settings category, something I don't have the luxury of.

`main/index.js lines 29-34` (edited for clarity)
```js
goosemodScope.settings.createItem('Custom CSS', [
    `(v${version})`, { type: 'custom', element: () => {
        // code that makes and returns the node they 
        // want to be in the settings menu
```     

Inside that, they initialize the div that will contain their editor with some css.

`main/index.js lines 35-43` (edited for clarity and briefness)
```js
// add div for ace to be in
const el = document.createElement('div');
// give it an id so ace can see it
el.id = 'gm-editor';
// size it to user's window
el.style.width = '90%';
el.style.height = '85vh';
// set its default value
el.innerHTML = css;
```

Then, they do 2 things:
- start a new thread that waits 10ms, then initializes Ace
- return the div

This is presumably because they cant initialize the editor until the div that it goes in exists.

```js
// instantly ran async function (async function() {..})();
(async function() {
    // wait 10 milliseconds (they wrote a method for this earlier)
    await sleep(10);

    // point ace to the id we gave our div
    const editor = ace.edit('gm-editor');
    const session = editor.getSession();
    // configure ace's ????, syntax highlighting, theme
    session.setUseWorker(false); // Tell Ace not to use Workers
    session.setMode('ace/mode/css'); // Set lang to CSS
    editor.setTheme('ace/theme/monokai'); // Set theme to Monokai
    // equivalent to addEventListener("input")
    session.on('change', () => {
        // some magic with localstorage we will get into later
        const val = session.getValue();
        css = val;
        updateCSS(val);
    });
})();

return el;
```

We wont need this async function because we can just add the div to the document then initialize the editor.

## Implementing what we learned
From what I read, it seems like initializing Ace will be pretty easy once we get it imported.

### Importing ace
Attempting to do exactly what GooseMod did would be nice, but unfortunately it seems Slack has blocked `eval()` on untrusted text. 

Attempting to run in the console
```js
eval(await (await fetch(`https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js`)).text()); // Load Ace main
```
gives us this error

```
Uncaught EvalError: Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script
```

I next thought to try and append a `<script>` to the header 

```js
var script = document.createElement('script');
script.type = 'text/javascript';
script.src = "https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js";
document.head.appendChild(script);
```

Attempting to run this gives us a much more clear and descriptive error:

![Refused to load the script 'link to script' because it violates the following Content Security Policy directive: "script-src 'self' 'wasm-eval' 'wasm-unsafe-eval' https://*.sdkassets.chime.aws https://*.slack.quip.systems https://quip-cdn.com https://slack-prod.qvpc-cdn.com https://a.slack-edge.com/ https://b.slack-edge.com/](https://i.imgur.com/Jy3AH86.png)

What I think this means is that scripts are only allowed from URLs that match one of the following:
- `https://*.sdkassets.chime.aws`
- `https://*.slack.quip.systems`
- `https://quip-cdn.com`
- `https://slack-prod.qvpc-cdn.com`
- `https://a.slack-edge.com/`
- `https://b.slack-edge.com/`

Then, I thought could just add the URL for our libraries to the `scripts=[]` argument of our python injecting code!

```py
inject(slack_location, devtools=True, timeout=600, scripts=[
    # libraries
    "https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js",
    "https://ajaxorg.github.io/ace-builds/src-min-noconflict/theme-monokai.js",
    "https://ajaxorg.github.io/ace-builds/src-min-noconflict/mode-css.js",

    # our injected code
    inject_location
]) 
```
But no, our Electron Inject expects only actual files to be passed into the scripts.

![FileNotFoundError: [Errno 2] No such file or directory: 'https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js'
](https://i.imgur.com/4LsiVHi.png)

### Improving our inject script

Given this, I spent some time adding dynamic library downloading to our launch script.

Here was the basic idea in pseudocode

```
for each library file we need:
    check if its already downloaded to a libs folder
    if not
        download it
        move it to the libs folder
    add path to lib to the scripts argument 
```

Im not going to go over this line by line, but python reads pretty well and its well commented.

```py
# scripts we will inject are added to this array
scripts = []
# get the directory this python code is in
# or the Current Working Directory
cwd = getcwd()
# make the libs folder if it doesn't exist
libsFolder = f"{cwd}/libs"
if not exists(libsFolder):
    makedirs(libsFolder)
# array of libraries to install
libURLs = [
    "https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js",
    "https://ajaxorg.github.io/ace-builds/src-min-noconflict/theme-dracula.js",
    "https://ajaxorg.github.io/ace-builds/src-min-noconflict/mode-css.js"]
for libURL in libURLs:
    # gets the last substring that follows a /
    fileName = libURL.split("/")[-1]
    print(f"\nGetting library file: {fileName}")

    # check if we have already downloaded it
    if exists(f"libs/{fileName}"):
        print(f"\tfound {fileName} in libs")
    else:
        print(f"\t{fileName} not in libs, downloading")
        # download it
        download(libURL)
        # move it into libs
        rename(f"{cwd}/{fileName}", f"{libsFolder}/{fileName}")
    # add path to file to list of scripts to inject
    # we use realpath() here to get the fill directory to the file
    # because sometimes electron inject gets screwy with relative files
    scripts.append(f"{libsFolder}/{fileName}")
# add our own code as the last entry in scripts
# so all the scripts are loaded once our code runs
scripts.append(f"{cwd}/inject.js")
```

### Making our editor with Ace
While we couldn't use GooseMod's code directly for it's imports, its code for setting up ace can practically be copy pasted in place of our old textarea code.

Given we are using that part of their code in full, I have to include a license. 

I didn't like how much space adding the MIT license took up in my code, so I asked the developer who wrote the code if I actually needed to put it there:

![Ducko — looks cool, reminds me of a thing i did ages ago just put link to original source as a comment probably | CodeF53 — alright, thanks! | Ducko — and/or include original mit license, up to you it's kinda needed but also not so shrug](https://i.imgur.com/ovY78Kh.png)

Given this answer, I decided to include this comment.

```js
/*
Based on GooseMod CustomCSS, which is under the MIT License
https://github.com/GooseMod-Modules/CustomCSS/blob/64969856598a2cc2980988046e0ff266d64fa943/index.js#L35-L65
*/
```

So what did was changed other than variable/method names?

Changed the width/height css to the css I wrote for the old editor. 


```js
cssEditor.style.width = "100%";
cssEditor.style.height = "calc(100% - 0.5rem)";
// set its default value
cssEditor.innerHTML = getCustomCSS();
```

We added our old code to add the editor to settings content pane directly after we initialize it.

```js
// add editor to settings content pane
document.querySelector(".p-prefs_dialog__panel").replaceChildren(cssEditor)
```

Because we add the editor to the dom instantly, we don't need the async function waiting stuff.

This bit also had theme changed from whatever ugly thing they were using before to dracula.

```js
// point ace to the id we gave our div
const editor = ace.edit('slackMod-editor');
const session = editor.getSession();
// configure syntax highlighting, theme
session.setMode('ace/mode/css'); // Set lang to CSS
editor.setTheme('ace/theme/dracula'); // Set theme to Dracula
// when we change the content of it, update css
session.on('change', () => {
    updateCustomCSS(session.getValue());
});
```

![Slack Preferences Screen with Custom CSS tab selected with nice syntax highlighted editor](https://i.imgur.com/Ail4shD.png)

## Making CSS Persistent
Looking over GooseMod's Custom CSS code helped a lot with improving our editor, but there didn't seem to be any good hints towards what to do for persistent saving.

### Finding what to use

I asked for help with this in the discord and got a single word answer

![SmolAlli — localstorage](https://i.imgur.com/YuSmVcd.png)

Looking into that, I found [Mozilla's documentation on it](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage). Looking at the examples, it should be as easy as:

```js
// get
localStorage.getItem("slackMod-CSS")

// set
window.localStorage.setItem("slackMod-CSS", css);
```

So we want to have a default value for our custom css, so I did a quick check to see what it returns when a value hasn't been assigned yet.

![>>window.localStorage.getItem("test") -> null](https://i.imgur.com/KZssU7o.png)

### Using it
I started by adding these to our pre-existing get/set methods:
```js
// method to quickly change css
const updateCustomCSS = newCSS => { 
    // update in storage
    window.localStorage.setItem("slackMod-CSS", newCSS);
    // update currently applied CSS
    document.querySelector("#SlackMod-Custom-CSS").innerHTML = "" 
    document.querySelector("#SlackMod-Custom-CSS").appendChild(document.createTextNode(newCSS)); 
}
// method to quickly get inner css
const getCustomCSS = () => { 
    return window.localStorage.getItem("slackMod-CSS")
}
```

I then replaced where we set the default contents of our CSS with the following
```js
if (window.localStorage.getItem("slackMod-CSS") == null) {
    // use default CSS
    styleSheet.innerText = "/*Write Custom CSS here!*/"
    window.localStorage.setItem("slackMod-CSS", "/*Write Custom CSS here!*/")
} else {
    // get saved CSS
    styleSheet.innerText = window.localStorage.getItem("slackMod-CSS")
}
```

That worked!

## Now What?
Given my list of issues from last blog, here is what I have left to do:
- Selecting custom css in preferences then selecting a different category leads to an issue
    - I still have absolutely no clue how to fix this, I have spent a solid 3 hours trying different things
- Injecting is manual and hardcoded to the user's system
    - I think I will make my next blog on this, followed by a tutorial on how to install it on your own slack!