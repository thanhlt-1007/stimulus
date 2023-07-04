# HANDBOX

## Preface: The Origin of Stimulus

We write a lot of Javascript at [37signals](https://37signals.com/), but we don't use it to create "Javascript applications" in the contemporary sense. All our applications have server-side rendered HTML at their core, then add sprinkles of Javascript to make them sparkle.

This is the way of the [majestic monolith](https://m.signalvnoise.com/the-majestic-monolith/). 37signals runs across half a dozen platforms, including native mobile apps, with a single set of controllers, views, and models created using Ruby on Rails. having a single, shared interface that can be updated in a single place key to being able to perform with a small team, despite the many platforms.

It allows us to party with productivity like days of yore. A throwback to when a single programmer could make rapacious progress without getting stuck in layers of indirection or distributed system. A time before everyone thought the holy grail was to confine their server-side application to producing JSON for a JavaScript-based client application.

And it's also not to say that the proliferation of single-page Javascript applications hasn't brought real benefit. Chief amongst which has been faster, more fluid interfaces set free from the full-page refresh.

We wanted Basecamp to feel like that too. As though we had followed the herd and rewritten everythong with client-side rendering or gone full-native on mobile.

This desire led us to a two-punch solution: Turbo and Stimulus.

### Turbo up high, Stimulus down low

Before I get to Stimulus, our new modest Javascript framework, allow me to recap the proposition of Turbo.

Turbo descends from an approach called [pjax](https://github.com/defunkt/jquery-pjax), developed at Github. The bassic concept remains the same. The reason full-page reffreshes often feel show is not so much because the browser has to process a bunch of HTML sent from a serrver. Browsers are really gooog and really fast at that. And in mosst casses, the fact thay an HTML payload tends to be larger than a JSON payload doesn;t matter either (especially with gzipping). No. the reason is that CSS and Javascript has to be reinitialized and reapplied to the page again. Regardless of whether the files themselves are cached. Thif can be pretty slow if you have a fair amount of CSS and Javascript.

To get around this reinitialization, Turbo maintains a persistent process, just like single-page applications do. But largely an invisible one. It intercepts links and load new pages via Ajax. The server still returns fully-formed HTML documents.

This strategy alone can make most actions in most applications fell really fast (if they're able to return server response in 100-200ms, which is eminently posible with catching), For Basecamp, it speed up the page-to-page transitiom by ~3X. It gives the application that feel of responsiveness and fluidity that was a massive part of the appeal for single-page applications.

But Turbo alone is only half the story. The coarsely grained one. Below the grade of a full page change lies all the fine-grained fidelity within a single page. The behavios that shows and hides elements, copies content to a clipbpard, adds a new todo to a list, and all the other interactions we associate with a modern web application.

Prior to Stimulus, Basecamp used a smarttering of different styles and petterns to apply these sprinkles. Some code wwas just a pinch of jQuery, some code was a similarly sized pinch of vanilla Javascript, and some again was larger obejct-oriendted subsystems. They all usually worked off explicit event handling hanging of a `data-behavior` atribute

While it was easy to add new code like this, it wasn't comprehensive solution, and we had too many in-house styles and patterns coexisting. That mafe it hard to reuse code, and it made it hard for new developers to learn a consistent approach.

### The three core concepts in Stimulus

Stimulus rolls up the best of those patterns into a modest, small framework revolving around just three main concepts: Controllers, actions and targets.

It's designed to read as a progressive enhancement when you look at the HTML it's addressing. Such that you can look at a single template and know which behavior is acting upon it. Here's an example:

```
<div data-controller="clipboard">
  PIN: <input data-clipboard-target="source" type="text" value="1234" readonly>
  <button data-action="clipboard#copy">Copy to Clipboard</button>
</div>
```

You can read that and have a pretty good idea pf wjat's going on. Event without knowing anythong about Stimulus or looking that the controller code itself. It's almost like pseudocode. That's very different from reading a slice of HTML that has an external Javascript file apply event handlers to it. It also maintains the separation of concerns that has been lost in many contemporary JavaScript frameworks.

As you can see, Stimulus doesn't bother itself with creating the HTML. Rather, it attaches itself to an existing HTML document. The HTML is, in the majority of cases, rendered on the server either on the page load (first hot or via Turbo) or via an Ajax request that changes the DOM.

Stimulus is concerned with naipulating this existing HTML document. Sometimes that means adding a CSS class hides an element or animated it or highlights it. Sometimes it means rearranging elements in groupings. Sometimes it means manipulating the content of an element, like we transform UTC times that can be cached into local times that can be displayed.

There are cases where you'd want Stimulus to create new DOM elements, and you's definitely free to do that. We might even add some sugar to make it easier in the future.But it's the minority use case. The focus is on manipulating, not creating elements.

### How Stimulus differs from mainstream JavaScript frameworks

This makes Stimulus very different from the majority of contemporary JavaScript frameworks. Almost all focused on turning JSON into DOM elements via template language of some sort. Many use these frameworks to birth an empty page, which is then filled exclusively with elements created through this JSON-to-template rendering.

Stimulus also differs on the question of state. Most frameworks have ways of maintaining state within JavaScript objects, and then render HTML based on that state. Stimulus is the exact opposite. State is stored in the HTML, so that controllers can be discarded between page changes, but still reinitialize as they were when the cached HTML appears again.

It really is a remarkably different paradigm. One that I'm sure many veterent JavaScript developers who've been used to work with ontemporary frameworks will scoff at. And hey, scoff away. It you're happy with the complexity and effort it takes to maintain an application within the saelstrom of, say React + Redux, then Turbo + Stimulus will not appeal to you.

If, on the other hand, you have nagging sense that what you're working on does not warrant the intense complexity and application separation such contemporary techniques imply, then you're likely to find refuge in our approach.

## <u>1. Introduction</u>

### a. About Stimulus

Stimulus is a JavaScript framework with modest ambitions. Unlike other front-end frameworks, Stimulus is designed to enhance static or server-rendered HTML - the "HTML you already have" - by connnecting JavaScript objects to elements on the page using simple annotations.

These JavaScript objects are called controllers, and Stimulus continuosly monitors the page wwaiting for HTML `data-controller` attributes to appear. For new each attribute, Stimulus looks at the attribute's value to find a corresponding controller class, create a new instance of that class, and connects it to the element.

You can think of it this way: jusst like the `class` attribute í a bridge connecting HTML to CSS, Stimulus's `data-controller` attribute is a bridge connecting HTML to JavaScript.

Aside from controllers, the three other major Stimulus concepts are

- `actions`, which connect controller methods to DOM events using `data-action` attributes

- `targets`, which locate elements of significance without a controller.

- `values`, which read, write, and observe data attributes on the controller's element

Stimulus's use of data attributes helps seperate content from behavior in the same way CSS seperates content from presentation. Futher, Stimulus's conventions naturally encourage you to group related code by name.

In turn, Stimulus helps you build small, reusable controllers, giving you just enough structure to keep your code from devolving into "JavaScript soup"

## <u>2. Hello, Stimulus</u>

### a. Prerequisites

To follow along, you'll need a running a copy of the [stimus-starter](https://github.com/hotwired/stimulus-starter) project, which is a preconfigured blank slate for exploring Stimulus.

We recommended remixing stimulus-starter on Glitch so you can work entirely in your browser without installing anything

Or, if you'd prefer to work from the comfort of your own text editor, you'll need to clone and set up `stimulus-starter`

```
git clone https://github.com/hotwired/stimulus-starter.git
cd stimulus-starter
yarn install
yarn start
```

Then visit `http://localhost:9000` in your browser.

Note that the `stimulus-starter` project use the [Yarn package manager](https://yarnpkg.com/) for dependency management, so make sure you have that installed first.

## <u>3. Building Something Real</u>

## <u>4. Designing For Resilience</u>

## <u>5. Managing State</u>

## <u>6. Working With External Resources</u>

## <u>7. Installing Stimulus in Your Application</u>
