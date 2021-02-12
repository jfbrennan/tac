You’re probably not in the mood for a new CSS methodology, but the web platform has progressed enough that it's time to at least consider another approach.

The methodology is called **Tag, Attributes, then Classes** (TAC) and it's quite simple. 

It was first developed at Expedia while building a framework-agnostic design system used by globally-distributed teams. It's since been refined and used at other companies, including a small AI startup, which means TAC scales up and down (more on that later).

Before we start, let's remember there are no silver bullets in software design, only tradeoffs. As for TAC, it optimizes for solving these four big problems of UI development:
1. Generic repeated styles
1. Components
1. Evolution and maintenance
1. Sharing the code

Application-centric problems, like routing or state, are outside its scope. TAC is UI-centric.

For future reference, there is an outline of the process all the way at the bottom.

Let's begin!

---

## 1. Generic repeating styles
All web designs include some amount of repeating styles. Most designs repeat a lot. These styles are as granular and tangible as color and font size, but also conceptual like layout and other design patterns. Most CSS methodologies find and implement these common styles as reusable **classes** and to an extent TAC includes this technique as well. 

To demonstrate, here's a simple example of abstracting parts of an imaginary design into three classes:
```css
.bold {
  font-weight: bold;
}

.active {
  color: #rrggbb;
  border: 1px solid #rrggbb;
}

.sidebar {
  display: flex;
  flex-direction: column;
  width: 280px;
  padding: 12px;
  border-right: 1px solid #cdcdcd;
  background-color: #ffffff;
}
```
In practice, methodologies like BEM or Tailwind produce code that is more refined and organized, but the important point here is their mode of implementation is _only and always_ classes. That’s different than TAC.

### Utility classes only
The only style from above that TAC would implement with a class is the bold font. The others have a semantic meaning. For example, `active` means something more than the blue text and blue border the designer gave it. It's special. The `bold` class on the other hand means - and does - exactly what it says. It's utilitarian and those are the only classes in TAC.

This doesn't mean you won't have a lot of classes though. A typical design will necessitate dozens of utility classes. If you're creating a design system, you'll easily need more than a hundred utility classes.

Utility classes should do one thing and one thing only, like `bold`. You'll _combine_ these classes to produce the required design, so it's important to take naming into consideration.

TAC doesn't prescribe any hard rules for naming utility classes, but offers these tips:
- Single character abbreviations are not intuitive, e.g. `p` could mean "padding" or "position", "perspective", "page", "pointer"
- Prefix related classes, e.g. `bold` should really be `font-bold` along with `font-italic`, `font-reg`, `font-light`, etc.
- Be consistent, e.g. size is relevant to text, space, breakpoints and more, so use a consistent size language like `sm`, `md`, `lg`
- In order to avoid confusion don't use other methodology's conventions, `lowercase-with-dashes` is all you need

### Find your design tokens
Some styles are so simple they amount to no more than a single value. These are known as _design tokens_. As you implement your utility classes and components (more on those in the next section) you'll start to notice they share a lot of the same tokens. To make your code DRY those values need to be centralized and reused and so TAC requires all **design tokens to be implemented as [CSS Custom Properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)**. 

So, in the example above we would pull common values into custom properties and then reference those properties like this:
```css
:root {
  --color-blue: #rrggbb;
  --color-gray: #cdcdcd;
  --size-sm: 12px;
}

.font-bold {...}

.active {
  color: var(--color-blue);
  border: 1px solid var(--color-blue);
}

.sidebar {
  ...
  padding: var(--size-sm);
  border-right: 1px solid var(--color-gray);
  ...
}
```
Depending on your project, you will end up creating a small bunch of custom properties for the sake of keeping or your code DRY, or you will build out an official collection of shared design tokens with everything from colors and standardized sizes to typefaces, shadows, borders, and much more.

So that's how TAC solves for generic repeated styles. 

Let's recap.
1. Create utility classes that do one generic thing
2. Use custom properties for design tokens  

## 2. Components
As a UI engineer finds and implements repeated styles, they will start to notice small to medium-sized sections of the design that have a distinct purpose of their own. These sections are more than generic styles, they are components.

Sidebar from the classes above is a component. 

Active is not. It's a _variation_ of a component. 

There's lots of components. Some components are interactive, some deal with layout, and some are for communication. Some are big, some are small, some do a lot like map, and some do nothing like icon. Some are product-agnostic (everybody has buttons) and some are unique to your product. TAC covers all of the above! 

The first step in building a component is defining its public API. This is the code you write when you want to create an instance of the component. There’s so many ways this can be done, but TAC leverages the HTML constructs of tag, attributes, and nesting. No classes! 

A component may have variations of itself, like a large and small version, and components usually get used more than once in the design, but that's not always true. Buttons, for example, often have primary, secondary, and other variations and get used many times in the design. Sidebar on the other hand, will get used once and might not have any variations.

The utility classes only pattern, popularized by Tailwind, fails to solve for components in any substantial way. There’s no component definition or API, no reusability, no encapsulation.

BEM, SMACSS, and others begin to solve for components, but despite their popularity they can result in ugly code with a poor API. They also can’t evolve! Here’s a comparison of Tailwind, BEM, OOCSS and SMACSS implementations of the sidebar component:

Here’s their buttons:
```html
<button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">Label</button>
<button class="btn btn--primary">Label</button>
<button class="btn btn-primary">Label</button>
<button class="primary">Label</button>
```

TAC approaches component development very differently than other methodologies. Let's look at the process. 

### 1) Start with the component's tag
The first rule of building a component is whenever HTML provides a standard tag for a component, use it. So in the case of button we already have a tag:
```html
<button>
```
Some readers are probably thinking, _"Not so fast, what about links styled as buttons?"_ I'll get to that.

When HTML does not provide a tag, create your own that meets these requirements:
1. Has a **custom tag prefix**. A prefix avoids conflicts with future HTML tags and other custom tags. Another purpose of this prefix that is essential to the TAC methodology is to prepare our component for the possibility of evolving into a JavaScript-enabled Custom Element. For these examples I'll use a `c-` prefix, which is short for...idk, it was the first key I noticed. In real life I use [`m-`](https://m-docs.org). Pick whatever you want, but be mindful of other custom tags that may already be using the same "namespace".
1. Has a **short but meaningful name**. Feel free to combine or abbreviate words like HTML does, e.g. "option group" as `optgroup` and "division" as `div`, but don't use single characters like `a` for "anchor".

Unlike button, sidebar has no standard element to leverage, so we'd define a new `c-sidebar` tag for that component. 

Ok, so far we have two components. One is standard HTML, the other isn't but looks like it could be:
```html
<button>
<c-sidebar>
``` 

And check it out, we're already producing better code:
```html
<c-sidebar>
  <p>Hello, Sidebar.</p>
</c-sidebar>

vs.

<div class="sidebar">
  <p>Hello, Div. I mean Class. Wait... Hello, Sidebar.</p>
</div>
```
The next step is styling.

### 2) Add a core set of component styles
We now add core styles to these components:
```css
button {
  Size, padding, margin, whatever...
}

c-sidebar {
  Background, border, width, whatever...
}
```

If you're uncomfortable with seeing non-standard HTML tags in your markup and stylesheet, don't be. It's not widely known, but browsers will parse these tags into real DOM elements and style them with any matching styles. This is not a trick or a hack. This is perfectly valid code. Standards-based and dependency-free I might add!

One important styling problem that's solved by TAC is **specificity**. Here's how:

First, TAC is able to start with the lowest specificity first because its components use tags. Tag specificity is 0-0-1.

Second, a good component will be styled to such a degree that it is perfectly usable as-is, but flexible enough to be customized with utility classes. Classes can override any matching property of the tag because their specificity is 0-1-0.

And finally, component styles that should not be overridden are those that pertain to one of the component's variations and because attributes are used for variations, the tag+attribute selector combo beats classes 0-1-1 to 0-1-0. Nice!

Here's a simple example:
```html
This button's default padding is overridden by
the utility class, but its `disabled` text color is not.

<button disabled class="pad-all-lg txt-pink">Label</button>

<style>
button {
  padding: 6px;
  color: blue;
}

button:disabled {
  color: gray;
}

.pad-all-lg {
  padding: 32px;
}

.txt-pink {
  color: pink;
}
</style>
```

Any core component styles - those matching on the tag - that should not be customizable can appropriately use `!important`. 

This approach is more capable overall at managing specificity than class-based methodologies.

Next, we begin to implement whatever variations the component has.

### 3) Define the component's variations
Most, but not all components are more than a static one-dimensional design. They have variations that include things like state (e.g. open, disabled, selected), type, behaviors (e.g. autofocus, autodismiss, draggable), metadata, children, and more.

To implement the primary and secondary variations of our `<button>` component, TAC again looks to the HTML standards and in this case finds nothing. But it doesn’t rush to classes! The reason classes are not used is the same reason HTML itself does not use them: primary and secondary variations, like the `disabled` state variation, are part of the button’s semantic meaning, and that's exactly what attributes are for.

When there is no standard attribute to use, **you make your own**. This is where TAC gets seriously fun!

As you look at the component and begin to define all of its different variations (state, type, children, etc.), take your time and look to HTML for guidance. These attributes are the component's public API and so they need to be good. Good attributes are intuitive and match developers expectations. If your attributes could be confused with real HTML attribute you’re doing good.

Ok, back to button. The primary and secondary variations communicate different levels of priority or importance of the button, so next thing we do is prototype a bunch of words for this attribute:
```html
<button priority="primary">
<button pri="primary">
<button importance="primary">
<button level="primary">
<button heirarchy="primary">
<button order="primary">
```
A good practice is to turn to the dictionary and ask, "Is there a word for primary, secondary, tertiary?" and you'll usually find a good answer: [ordinal number words](https://en.wikipedia.org/wiki/Ordinal_numeral), in this case.

So, let's try that too:
```html
<button ordinal-number-word="primary">
<button ordinalnum="primary">
<button onw="primary">
<button ordinal="primary">
<button ord="primary">
```
Like coming up with good class names, there's no right or wrong answer. You just have to try a bunch until something sticks. Ask others if they find it intuitive. My favorite example is an alert component feature that automatically dismisses the alert. I quickly think of input's `autofocus` and `autocomplete` attributes, and because TAC says to follow existing nomenclature, I came up with `autodismiss`.

Here’s a few more examples to help demonstrate:
```html
<c-badge count="5"></m-badge>

<m-alert type="success" autodismiss>...</m-alert>

<m-loader loading>Saving...</m-loader>

<m-row>
  <m-col span="3 md-5 sm-12" order="sm-2">
    ...
  </m-col>
  <m-col span="9 md-7 sm-12" order="sm-1">
    ...
  </m-col>
</m-row>
```

Thinking of attributes for variations and using different patterns, like boolean attributes or attributes with multiple values, is a creative process you'll enjoy. You’ll find there’s a lot of untapped power in [attribute selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors).

One more thing. The links-that-need-to-look-like-buttons dilemma from earlier. It's the same process of turning to HTML for answers and in this case we get a good answer: `role="button"`. All rules with the `button` selector simply include `a[role=button]`. 

With custom or standard tags and attributes you never ever need to resort to classes to define your component. Leave classes for generic styles only!

### The code speaks for itself
Tabs following BEM ([from Material](https://material.io/develop/web/components/tabs/tab-bar))
```html
<div class="mdc-tab-bar" role="tablist">
  <div class="mdc-tab-scroller">
    <div class="mdc-tab-scroller__scroll-area">
      <div class="mdc-tab-scroller__scroll-content">
        <button class="mdc-tab mdc-tab--active" role="tab" aria-selected="true" tabindex="0">
          <span class="mdc-tab__content">
            <span class="mdc-tab__text-label">A</span>
          </span>
          <span class="mdc-tab-indicator mdc-tab-indicator--active">
            <span class="mdc-tab-indicator__content mdc-tab-indicator__content--underline"></span>
          </span>
          <span class="mdc-tab__ripple"></span>
        </button>
        <button class="mdc-tab mdc-tab--active" role="tab" aria-selected="true" tabindex="0">
          <span class="mdc-tab__content">
            <span class="mdc-tab__text-label">B</span>
          </span>
          <span class="mdc-tab-indicator mdc-tab-indicator--active">
            <span class="mdc-tab-indicator__content mdc-tab-indicator__content--underline"></span>
          </span>
          <span class="mdc-tab__ripple"></span>
        </button>
      </div>
    </div>
  </div>
</div>
```

Tabs following OOCSS ([from Bootstrap](https://getbootstrap.com/docs/5.0/components/navs-tabs/#javascript-behavior))
```html
<ul class="nav nav-tabs" id="tabs" role="tablist">
  <li class="nav-item" role="presentation">
    <a class="nav-link active" id="a-tab" data-toggle="tab" href="#a" role="tab" aria-controls="a" aria-selected="true">A</a>
  </li>
  <li class="nav-item" role="presentation">
    <a class="nav-link" id="b-tab" data-toggle="tab" href="#b" role="tab" aria-controls="b" aria-selected="false">B</a>
  </li>
</ul>
```
Tabs following TAC ([from M-](https://m-docs.org/tabs), a11y is built in)
```html
<m-tabs scrollable>
  <m-tab>A</m-tab>
  <m-tab>B</m-tab>
</m-tabs>
```

## 3. Evolution and maintenance
Perhaps the biggest downside of other CSS methodologies is their inability to evolve components beyond the original styles-only implementation. Yes, they work fine when adding or modifying variations that only require CSS, but the moment a component built with classes requires JavaScript it's back to the drawing board. That's not the case for TAC.

As we learned during the tag definition step, TAC prepares components for this situation. Let's see what happens when our styles-only `c-sidebar` component gets new features that require JavaScript...

**The current sidebar**
```html
<c-sidebar>
  <img src="..." alt="..."/>
  <nav>
    <a href="...">Link</a>
    <a href="...">Link</a>
    <a href="...">Link</a>
  </na>
  <small>Copyright 2021</small>
</c-sidebar>

<style>
c-sidebar {
  display: flex;
  flex-direction: column;
  width: 280px;
  padding: var(--size-sm);
  border-right: 1px solid var(--color-gray);
  background-color: white;
}

c-sidebar > img {
  width: 80px;
  margin: 0 auto;
}

c-sidebar > nav > a {
 display: block;
}

c-sidebar > small {
  text-align: center;
  color: var(--color-gray);
}
</style>
```

It's a vertical container with some core styles for itself and some expected child elements. 

Now let's say an updated sidebar design includes a button that toggles sidebar open or closed. That's where the need for JavaScript comes in and because our components are tag-based, we get to turn to Custom Elements and define an element for our tag:

```javascript
customElements.define('c-sidebar', class extends HTMLElement {
  constructor() {
    super();
  }

  // Create the toggle button and append it
  connectedCallback() {
    this.toggleBtn = document.createElement('button');
    this.toggleBtn.textContent = 'Close';
    this.toggleBtn.classList.add('pad-all-xs', 'txt-sm', 'pos-absolute', 'pin-t', 'pin-r');
    this.toggleBtn.addEventListener('click', this.toggle);
    this.append(this.toggleBtn);
  }

  // Observe the closed attribute
  static get observedAttributes() {
    return ['closed'];
  }

  // Dispatch "toggle" event when sidebar is opened or closed
  attributeChangedCallback(name, oldVal, newVal) {
    if (name === 'closed') {
      this.dispatchEvent(new CustomEvent('toggle'));
    }
  }

  get closed() {
    return this.hasAttribute('closed');
  }

  set closed(isClosed) {
    isClosed ? this.setAttribute('closed', '') : this.removeAttribute('closed');
  }

  // Toggles closed state and updates button label
  toggle() {
    this.closed = !this.closed;
    this.toggleBtn.textContent = this.closed ? 'Open' : 'Close';
  }

});
```
In the interest of time, let's just make the sidebar's width abruptly change based on the `closed ` state of the sidebar:
```css
c-sidebar {
  ...
  width: 280px;
  ...
}

c-sidebar[closed] {
  width: 80px;
}
```
Ok, so now sidebar will have 1) a button that toggles the sidebar, 2) the button label will change automatically, and 3) there's a custom event to listen to using whatever method the application prefers to bind events (e.g. `ontoggle`, `v-on:toggle`, `sidebar.addEventListener('toggle')`, etc.).

**What is most important here though is this:**
_After the evolution to Custom Element all existing instances of this sidebar component will continue to function without any breaking changes or shifting paradigms requiring the developer to refactor. Sidebar just works!_

It's like we've added a V8 engine to this bicycle of a component and the rider never had to take their foot off the pedals.

And the same can be true in the other direction: if all JavaScript-enabled variations of the component are no longer needed, the component can safely scale down to a styles-only custom tag.

The freedom to seamlessly upgrade or downgrade components at-will is very powerful, especially if you're building a shared design system, which undergo a lot of iteration. This is why you **must start with a prefixed tag**.

Maintenance is the most expensive part of software. TAC is cheaper to maintain because of its simplicity and inherent longevity by aligning with HTML.

## 4. Sharing the code
At this point the build is done, but what good is it if it can't be shared? Here's how TAC makes sharing easy.

### Design systems are a must
In the past, implementation was done in a vacuum. The code only worked for the original design. Over time web applications became more sophisticated and subject to more frequent iteration, which required processes to improve. The use of a shared design system has become the standard. TAC was developed while building several design systems at large and small companies.

### Dependencies are a mill stone
Even with a design system, truly shareable code cannot be achieved because all teams become dependent on the design systems underlying JavaScript framework, such is the case with Material-UI and React. If your whole organization is committed to one framework for all projects, then that’s not a big problem, but if you value the freedom to use other JavaScript frameworks, or need a different architecture, or have an existing collection of projects (e.g. legacy apps, apps from an acquisition, apps of various tech stacks, public website, landing pages, CMS-backed sites, web views, email templates), then it's not even feasible to standardize on one framework-dependent design system. 

Other CSS methodologies can't avoid this problem because they can't do components without a framework (e.g. Bootstrap's jQuery, React, Angular, etc. versions). TAC excels at being shared because it doesn't have dependencies.

### Web standards #FTW
Because TAC leverages web standards all the way from styles to static and dynamic components, it works with pretty much any web project old and new. Anything that supports HTML should work. Even some email clients, which are notoriously difficult and fussy, support custom tags (but not Custom Elements since there's no JavaScript). This breadth of support is especially good for large companies and agencies because they have such a wide variety of projects. I've seen cross-team adoption happen firsthand because of this one reason. No matter how cool something is, it's of little use if it can't be shared.

### Adoption and limits
What good is a library if it can’t be adopted? 

A design system's value is tied to its adoption rate. The more use, the more value it provides and so TAC optimizes for ease and breadth of adoption and longevity by following standards. It stands to reason then, that the most valuable design systems will be those following the TAC methodology.

Let's end with what we started: there are no silver bullets.
TAC is just the UI layer. For some projects, that's more than enough. For modern web applications, frameworks like Vue, Riot, Svelte, and even React play a critical role, but with TAC **the role of frameworks narrows**. You no longer use these frameworks for UI components. Instead, you use them for _application structure and state management_. This visual helps explain:




## Review
**Tag, Attributes, and Classes** helps you solve the four main problem areas of implementing a design: generic repeating styles, components, evolution and maintenance, sharing the code.

If you want a quick reference (or got bored and just scrolled down here), there's a reference outline of TAC below. 

If you want to see a design system built following the TAC methodology, check out [M-](https://m-docs.org) (pronounced "em dash"). You can also read [10 Ways M- Raises the Bar for UI Libraries](https://dev.to/jfbrennan/10-ways-m-raises-the-bar-for-ui-libraries-2p4i) and follow [@realEmDash](https://twitter.com/realEmDash).
**TAC Outline**
1. Create utility classes that do one generic thing
  - Avoid single character abbreviations
  - Prefix related classes, e.g. `.txt-center`, `.txt-right`
  - The `.lowercase-dash-separated` convention is all you need
1. Define [custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties) for design tokens 
1. Use tags and attributes for components
  - Use up HTML, then imitate it
  - Define a custom tag prefix
  - Give tags short but meaningful names
  - Use attributes for component variations
  - Specificity scores increase in tag -> utility class -> tag+attribute order
1. Evolve your components with Custom Elements
  - Components always start life as a CSS-only custom tag
  - Upgrade to Custom Elements when JavaScript is required
  - Upgrading and downgrading is non-breaking
1. TAC is ideal for shared design systems
  - Framework-agnostic
  - Static sites, SPA, SSR, PWA - they're all supported
  - Value is in the adoption rate
