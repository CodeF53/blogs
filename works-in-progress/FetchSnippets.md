## Intro:
I have recently found myself writing the same code for fetch requests over and over again.

I remembered that snippets were a thing in VSCode, so I made a couple for fetch requests, after doing so I figured I should make a blog explaining how to make snippets.

## Adding custom snippets:
First, go into VSCode.

File > Preferences > Configure User Snippets
![File > Preferences > Configure User Snippets](https://cdn.discordapp.com/attachments/1006247423313719339/1017494054214324295/unknown.png)

Click "New Global Snippets file..."
![Mouse cursor over "New Global Snippets file..."](https://cdn.discordapp.com/attachments/1006247423313719339/1017494054541475911/unknown.png)

Give the file some appropriate name, I used `jsFetch.code-snippets`

Paste in some snippet code

```json
{
	"fetchAdd": {
		"scope": "javascript",
		"prefix": ["fetch-add"],
		"body": ["fetch(\"http://${1:url}\",{\n\tmethod: \"POST\",\n\theaders: {\n\t\t\"Content-Type\": \"application/json\"\n\t},\n\tbody: JSON.stringify(${2:object})\n}).then(r=>r.json()).then((data)=>{\n\t$3\n})"],
		"description": "adds a new object to a server at URL"
	},

	"fetchRemove": {
		"scope": "javascript",
		"prefix": ["fetch-remove"],
		"body": ["fetch(`http://${1:url}/${${2:id}}`,{\n\tmethod: \"DELETE\"\n}).then(r=>r.json()).then((data)=>{\n\t$3\n})"],
		"description": "removes an object from a server at URL/id"
	},

	"fetchPatch": {
		"scope": "javascript",
		"prefix": ["fetch-patch", "fetch-edit"],
		"body": ["fetch(`http://${1:url}/${${2:id}}`,{\n\tmethod: \"PATCH\",\n\theaders: {\n\t\t\"Content-Type\": \"application/json\"\n\t},\n\tbody: JSON.stringify(${3:patchObject})\n}).then(r=>r.json()).then((data)=>{\n\t$4\n})"],
		"description": "edits an object from a server at URL/id"
	}
}
```

Save

## Using Custom Snippets:
Start by typing the first few characters of the snippet you want to select:
![VSCode JS file, "fet" is typed, with suggestions "fetch","fetch-add","fetch-edit","fetch-patch","fetch-remove", "fetch" is selected](https://i.imgur.com/O7fHdrd.png)

Press the down arrow key till the snippet you want to use is highlighted:
![above image, but now "fetch-edit" is now selected](https://i.imgur.com/lHbgMNe.png)

Press tab:
![VSCode JS file, it now has a fetch edit statement with temporary texts: "url", "id", "patchObject", url is selected](https://i.imgur.com/1mXU8UX.png)

Type in the corresponding value of the currently selected text:
![above image, but the temporary "url" text now says localhost:3000](https://i.imgur.com/zjRzwCv.png)

Press tab again to go to the next temporary value, and repeat:
![gif of what was above, with user tabbing between fields and typing in data](https://cdn.discordapp.com/attachments/1006247423313719339/1017500282076270613/ezgif.com-gif-maker.gif)

## Understanding and Making your own snippets:
### Formatting code for use in snippets:
Start with the boilerplate code you want to make a snippet of with some example variables:
```js
fetch(`http://url/${id}`,{
    method: "PATCH",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify(patchObject)
}).then(r=>r.json()).then((data)=>{
    
})
```

Then, replace the variables you want with `${TABINDEX:NAME}`. TABINDEX here indicates what order you will end up tabbing between the variables, counting up from 1:

```js
fetch(`http://${1:url}/${${2:id}}`,{
    method: "PATCH",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify(${3:patchObject})
}).then(r=>r.json()).then((data)=>{
    
})
```

You can also make it so you can tab to a location with `$TABINDEX`:

```js
fetch(`http://${1:url}/${${2:id}}`,{
    method: "PATCH",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify(${3:patchObject})
}).then(r=>r.json()).then((data)=>{
    $4
})
```

Wrap the your code in quotes, and if necessary, use `\` to escape any special characters like `"` and `\`:
```json
"fetch(`http://${1:url}/${${2:id}}`,{
    method: \"PATCH\",
    headers: {
        \"Content-Type\": \"application/json\"
    },
    body: JSON.stringify(${3:patchObject})
}).then(r=>r.json()).then((data)=>{
    $4
})"
```

Replace tabs/indents with `\t`:
```json
"fetch(`http://${1:url}/${${2:id}}`,{
\tmethod: \"PATCH\",
\theaders: {
\t\t\"Content-Type\": \"application/json\"
\t},
\tbody: JSON.stringify(${3:patchObject})
}).then(r=>r.json()).then((data)=>{
\t$4
})"
```

Replace new lines with \n
```json
"fetch(`http://${1:url}/${${2:id}}`,{\n\tmethod: \"PATCH\",\n\theaders: {\n\t\t\"Content-Type\": \"application/json\"\n\t},\n\tbody: JSON.stringify(${3:patchObject})\n}).then(r=>r.json()).then((data)=>{\n\t$4\n})"
```

### Making Snippets

Snippets are formatted as JSON objects:

The outer key is the name of the snippet, this is what is used for the name shown on the right of VSCode's autofill:
```json
"fetchAdd" {
    ...
}
```

The `scope` key defines what languages VSCode will suggest the snippet in. This is a single string, with commas separating languages if nessesary:
```json
"fetchAdd" {
    "scope": "javascript,html",
    ...
}
```

The `prefix` key defines what you can type to have VSCode suggest the snippet, as well as what the snippet shows up as on the left side fo VSCode's autofill. This is an array of strings:
```json
"fetchAdd" {
    ...
    "prefix": ["fetch-add", "example-two"],
    ...
}
```

The `body` key defines the code the snippet makes, along with the order you tab through it. This is the formatted code we made earlier:
```json
"fetchAdd": {
    ...
    "body": ["fetch(\"http://${1:url}\",{\n\tmethod: \"POST\",\n\theaders: {\n\t\t\"Content-Type\": \"application/json\"\n\t},\n\tbody: JSON.stringify(${2:object})\n}).then(r=>r.json()).then((data)=>{\n\t$3\n})"],
    ...
},
```

The final key, `description` controls what text shows up when you click read more on the snippet in VSCode's autofill:
```json
"fetchAdd": {
    ...
    "description": "Example Text!"
}
```
![VSCode JS, mouse hovering over a snippet on the right side, cursor tooltip says "Read More"](https://i.imgur.com/R5u1Vfc.png)
![above picture, but read more has been clicked, toggling on a preview pane of the snippet, the top of the preview pane reads "Example Text! (Global User Snippet)"](https://i.imgur.com/tcvZfrs.png)

Your final snippet will look something like this:
```json
"fetchAdd": {
    "scope": "javascript",
    "prefix": ["fetch-add"],
    "body": ["fetch(\"http://${1:url}\",{\n\tmethod: \"POST\",\n\theaders: {\n\t\t\"Content-Type\": \"application/json\"\n\t},\n\tbody: JSON.stringify(${2:object})\n}).then(r=>r.json()).then((data)=>{\n\t$3\n})"],
    "description": "adds a new object to a server at URL"
},
```