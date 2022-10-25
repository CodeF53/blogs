## Intro
If you have ever made something with Create-React-App, you probably have gotten used to seeing `8 high severity vulnerabilities`.

![8 high severity vulnerabilities](https://i.imgur.com/IEIl3tV.png)

These problems never get solved by the devs of Create-React-App because these aren't actual vulnerabilities:

[*Maintainer of Create-React-App's response to github issue about the vulnerability*](https://github.com/facebook/create-react-app/issues/11647#issuecomment-1243863292)
> As often, it's not a real vulnerability. If you read the description, it explains that the attack vector is via network, but this is a build-time dependency that doesn't exist in built apps (the only dependency of a CRA app after build is React itself). As explained in #11174 we will not be addressing fake vulnerabilities.
>
> If this is frustrating to you, please express it to npm, who are responsible for this security theater. Some npm support channels:
> - https://github.com/npm/cli/issues
> - https://github.com/npm/rfcs/discussions
> - https://www.npmjs.com/support

Seeing these vulnerabilities is enough to annoy me into fixing it, even if unnecessary.

## Fixing the vulnerability:

This guide is based on [thomazcapra's solution](https://github.com/facebook/create-react-app/issues/12132#issuecomment-1130249584).


Because the vulnerability originate's from react's dependence on `@svgr/webpack`, fixing the issue is as simple as updating to a version that doesn't have it.

To do this, go to your `package.json`, and add the following:
```json
  "devDependencies": {
    "@svgr/webpack": "^6.2.1"
  },
  "overrides": {
    "@svgr/webpack": "$@svgr/webpack"
  }
```

Then, just delete your `node_modules` folder and `package-lock.json` and run `npm i`.

![found 0 vulnerabilities](https://i.imgur.com/yv15dyN.png)

Very satisfying.