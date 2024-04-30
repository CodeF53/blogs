The conventional way people write css is hard to read, maintain, and keep consistent.

## What's wrong with conventional selectors?
Conventional selectors bake in their entire component scope along-side what the element actually is.
This leads to very long, repetitive ids and classes in the DOM:
```html
<div id="componentName">
  <p>...</p>
  <div id="componentName_thingContainer">
    <span class="componentName_thingContainer_thing"></span>
    <span class="componentName_thingContainer_thing"></span>
  </div>
</div>
```

Along with repetitive selectors in the css:
```css
#componentName { /* top-level component styles */ }
#componentName_thingContainer { /* container styles */ }
.componentName_thingContainer_thing { /* entry styles */ }
```

If you ever see the same repetitive text while programming, you are probably doing something wrong.
So what can we do different?

## A DRYer method to selectors
Make your class/id just describe what the element is on its own:
```html
<div id="componentName">
  <p>...</p>
  <div id="thingContainer">
    <span class="thing"></span>
    <span class="thing"></span>
  </div>
</div>
```

Then, in css scope your styles from the top down, with descendant or child selectors:
```css
#componentName { /* top-level component styles */ }
#componentName > #thingContainer { /* container styles */ }
#componentName > #thingContainer > .thing { /* entry styles */ }
```

This still isn't very dry, so we can use CSS nesting with a preprocessor like Scss:
```scss
#componentName {
  // top-level component styles
  > #thingContainer {
    // container styles

    > .thing { /* entry styles */ }
  }
}
```

Using nesting for css selectors like this also means there is a set location for selectors, leading to a better experience while writing and maintaining:
- no more indecision, **"Where do I put the styles for search results?"**, the answer is always the descendent/child of whatever above it has an id/class
- no more searching, **"Where on earth are the styles for this input?"**, just follow the same path found inside the component `html`/`jsx`

## How to implement utility classes
Very often you will find yourself writing the same set of rules all over your codebase, for example:
```css
#searchResults { display: flex; flex-direction: column; }
#navContainer { display: flex; flex-direction: column; }
#options { display: flex; flex-direction: column; }
```

So just break out common styles that you use into classes in global scope:
```css
.row { display: flex; flex-direction: row; }
.col { display: flex; flex-direction: column; }
.spaceBetween { justify-content: space-between; }
.spaceAround { justify-content: space-around; }
.spaceEvenly { justify-content: space-evenly; }
```

### Super classes
If you see yourself using the same set of utility classes over and over together like this:
```html
<div id="confirmationModal" class="col gap1 centerChildren elv0 pad1 rad1"></div>
<section class="searchResult col gap1 centerChildren elv0 pad1 rad1"></section>
<div id="filterSelector" class="col gap1 centerChildren elv0 pad1 rad1"></div>
<div id="userSettings" class="col gap1 centerChildren elv0 pad1 rad1"></div>
```

Make a superclass that combines them:
```css
.panel {
  /* all of the rules contained within col, gap1, centerChildren, elv0, pad1, and rad1 */
  /* for extra points use something like scss's @mixin and @include */
}
```

## Utility classes > raw rules
When utility classes are available, use them to write the least code required to accomplish what you want.
This means favoring them over writing the rules within the component.

Instead of:
```html
<div id="componentName">
  <p>...</p>
  <div id="thingContainer">
    <span class="thing"></span>
    <span class="thing"></span>
  </div>
</div>
```
```scss
#componentName {
  // all of the rules contained within panel
  // a couple of rules that aren't in any utility class

  > #thingContainer {
    // all of the rules contained within col centerChildren gap1

    > .thing { /* entry styles */ }
  }
}
```

Do:
```html
<div id="componentName" class="panel">
  <p>...</p>
  <div class="col centerChildren gap1">
    <span class="thing"></span>
    <span class="thing"></span>
  </div>
</div>
```
```scss
#componentName {
  // a couple of rules that aren't in any utility class

  .thing { /* entry styles */ }
}
```

## Keep it organized
If you want other people to work on your project, you should make sure people can find what they are looking for.

Here is an example:
- `misc.scss`
  - stuff that doesn't fit elsewhere, like font declarations
- `components.scss`
  - default HTML components, like input/textarea/
  - large utility super classes, like `.panel` [from an earlier example](#super-classes)
- `layout.scss`
  - utility classes that control positioning
  - super classes for common layouts like pages with a sidebar
- `variables.scss`
  - standardized colors/sizes/shadows
- `util.scss`
  - utility classes that don't fit anywhere else
  - classes to apply variable values:
    - `.bg0` for the lowest background color
    - `.shadow` for applying a standard `box-shadow`