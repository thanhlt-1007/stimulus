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

```HTML
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

### b. About This Book

This handbox will guide you through Stimulus's core concepts by demonstrating how to write several fully functional controllers. Each chapter builds on the one before it; from start to finish, you'll learn how to:

- print a greeting addressed to the name in a text field.

- copy text from a text field to the system clipboard when a button is clicked.

- navigate through a slide show with multiple slides

- fetch HTML from the server into an element on the page automatically

- set up Stimulus in your own application

Once you've completed exercise here, you may find the [reference documentation](https://stimulus.hotwired.dev/reference/controllers) helpful for understading technical details about the Stimulus API.

Let's get started!

## <u>2. Hello, Stimulus</u>

### a. Prerequisites

To follow along, you'll need a running a copy of the [stimus-starter](https://github.com/hotwired/stimulus-starter) project, which is a preconfigured blank slate for exploring Stimulus.

We recommended remixing stimulus-starter on Glitch so you can work entirely in your browser without installing anything

Or, if you'd prefer to work from the comfort of your own text editor, you'll need to clone and set up `stimulus-starter`

```bash
git clone https://github.com/hotwired/stimulus-starter.git
cd stimulus-starter
yarn install
yarn start
```

Then visit `http://localhost:9000` in your browser.

Note that the `stimulus-starter` project use the [Yarn package manager](https://yarnpkg.com/) for dependency management, so make sure you have that installed first.

### b. It All Starts With HTML

Let's begin with a simple exercise using a text field and a button. When you click the button, we;ll display the value of the text field in the console.

Every Stimulus project starts with HTML. Open `public/index.html` and add the following markup jusst after the opening `<body>` tag:

```HTML
<div>
  <input type="text">
  <button>Greet</button>
</div>
```

Reload the page in your browser and you should sê the text field and button.

### c. Controllers Bring HTML to Life

At its core, Stimulus's purpose is tp automatically connect DOM elements to JavaScript objects. Those objects are called controllers.

Let's create our first controller by extending the framework's built-in `Controller` class. Create a new file named `hello_controller.js` in the `src/controllers/` folder. Then place the following code inside:

```JS
// src/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controllers {
}
```

### d. Identifiers Link Controllers With the DOM

Next, wwe ned to tell Stimulus how this controller should be connected to our HTML. We do this by placing an identifier in the `data-controller` attribute on our `<div>`:

```HTML
<div data-controller="hello">
  <input type="text">
  <button>Greet</button>
</div>
```

Identifiers serve as the link between elements and controllers. In this case, the identifier `hello` tells Stimulus to create an instance of the controller class in `hello_controller.js`. You can learn more about how automatic controller loading wworks in the [Installation Guide](https://stimulus.hotwired.dev/handbook/installing).

### e. Is This Thing On?

Reload the page in your browser and you'll see that nothing has changed. How to we know whether our controller is working or not?

One way is to put a log statement in the `connect()` method, which Stimulus calls each time a controler is connected to the document.

Implement the `connect()` method in `hello_controller.js` as follows:

```JS
// src/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  connect() {
    console.log("Hello, Stimulus!", this.element)
  }
}
```

Reload the page again and open developer console. You should see `Hello, Stimulus!` followed by a representation if our `<div>`.

### f. Actions Respond to DOM Events

Now let's see how to change the code so our log message appears when we click the "Greet" button instead.

Start by renaming `connect()` to `greet()`

```JS
// src/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  greet() {
    console.log("Hello, Stimulus!", this.element)
  }
}
```

We want to call the `greet()` method when the button's `click` event is triggered. In Stimulus, Controller methods which handle events are called action methods.

To connect our action method to the button's `click` event, open `public/index.html` and add a `data-action` attribute to the button:

```JS
<div data-controller="hello">
  <input type="text">
  <button data-action="click->hello#greet">Greet</button>
</div>
```

<u>Action Descriptors Explained</u>

The `data-action` value `click->hello#greet` is called an action descriptor. Thif particular descriptor says:

- `click` is the event name

- `hello` is the controller identifier

- `greet` is the name of the method to invoke.

Load the page in your browser and open the developers console. You should see the log message appear when you click the "Greet" button.

### g. Targets Map Important Elements To Controller Properties

We'll finish the exercise by changing our action to say hello to whatever name we've types in the text field.

In order to do that, first we need a reference to the input element inside our controller. Then we can read the `value` property to get its contents.

Stimulus let ú mark important elements as targets so we can easily reference them in the controller through corresponding properties. Open `public/index.html` and add a `data-hello-target` attribute to the input element:

```HTML
<div data-controller="hello">
  <input data-hello-target="name" type="text">
  <button data-action="click->hello#greet" >Greet</button>
</div>
```

Next, we'll create a property for the target by adding `name` to our controler's list of target definitions. Stimulus will automatically create a `this.nameTarget` property which returns the first matching target element. We can use this property to read the element's `value` and build our greeting string.

Let's try it out. Open `hello_controller.js` and update it like so:

```JS
// src/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controllers {
  static targets = ["name]

  greet() {
    const element = this.nameTarget
    const name = element.value
    console.log(`Hello, ${name}!`)
  }
}
```

Then reload the page in your browser and open the developer console. Enter your name in the input field and click the "Greet" button. Hello, wworld!

### h. Controllers Simplify Refactoring

We've seen that Stimulus controllers are instances of JavaScript classes whose methods can act as event handlers.

That means we have an arsenal of standard refactoring techniques at our disposal. For example, we can clean up our `greet()` method by actracting a `name` getter:

```JS
// src/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controllers {
  static targets = ["name"]

  greet() {
    console.log(`Hello, ${this.name}!`)
  }

  get name() {
    return this.nameTarget.value
  }
}
```

### i. Wrap-Up and Next Steps

Congratulations-you're just written your first Stimulus controller!

We've covered the framework;s most important concepts: controllers, actions and targets. In the next chapter, we'll see how to put those together to build a real-life controller taken right from Basecamp.

## <u>3. Building Something Real</u>

We've implemented out first controller and learned how Stimulus connects HTML to JavaScript. Now let's take a look at something we can use in a real application by recreating a controller from Basecamp.

### a. Wrapping the DOM Clipboard API

Scattered throughout Basecamp's IO are buttons like these:

![](https://stimulus.hotwired.dev/assets/bc3-clipboard-ui.png)

When you click one of these bittons, Basecamp copies a bit of text, such as a URL or an email address, to your clipboard.

The web platform has an [an API for accessing the system clipboard](https://www.w3.org/TR/clipboard-apis/), but there's no HTML element that does what we need. To implement a "Copy to clipboard" button, we must use JavaScript.

### b. Implementing a Copy button

Let's say we have an app which allows us to grant someone else access by generating a PIN for them. It would be convenient if we could display that generated PIN alongside a button to copy it to the clipboard for easy sharing

Open `public/index.html` and replace the contents of `<body>` with a rough sketch of the button:

```HTML
<div>
  PIN: <input type="text" value="1234" readonly>
  <button>Copy to Clipboard</button>
</div>
```

### c. Setting Up the Controller

Next, create `src/controllers/clipboard_controller.js` and add an empty method `copy()`:

```JS
// src/controllers/clipboard_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  copy() {
  }
}

```

Then add `data-controller="clipboard"` to the outer `<div>`. Any time this attribute appears on an element, Stimulus will connect an instance of our controller:

```HTML
<div data-controller="clipboard">
```

### d. Defining the Target

We;ll neef a reference to the text field so we can select its content before invoking the clipboard API. Add `data-clipboard-target="source"` to the text field:

```HTML
PIN: <input data-clipboard-target="source" type="text" value="1234" readonly>
```

Now add a target definition to the controllers so we can access the text field element as `this.sourceTarget`:

```JS
export default class extends Controller {
  static targets = ["source"]
}
```

> What's With That `static targets` Line ?

When Stimulus load your controller class, it looks for target name strings in a static array called `targets`. For each target name in the array, Stimulus adds three new properties to your controller. Here, our `"source"` target name becomes the following properties.

- `this.sourceTarget` evaluates to the first `source` target in your controller's scope. If there is no `source` target, accessing the property throws an error.

- `this.sourceTargets` evaluates to an array of all `source` targets in the controller's scope.

- `this.hasSourceTarget` evaluates to `true` if there is a `source` target or `false` if not.

You can read more about targets in the [reference document](https://stimulus.hotwired.dev/reference/targets).

### e. Connecting the Action

Now we're ready to hook up the Copy button.

We want a click on the button to invoke the `copy()` method in our controller, so we'll add `data-action="clipboard#copy"`:

```HTML
<button data-action="clipboard#copy">Copy to Clipboard</button>
```

> Common Events Have a Shorthand Action Notation

You might have noticed we've ommited `click->` from the action desciptor. That's because Stimulus defines `click` as the default event for actions on `<button>` elements.

Certain other elements have default events, too. Here's the full list

| Element             | Default Event |
|---------------------|---------------|
| a                   | click         |
| button              | click         |
| details             | toggle        |
| form                | submit        |
| input               | input         |
| input type="submit" | click         |
| select              | change        |
| textarea            | input         |

Finally, in our `copy()` method, we can select the input field's contents and call the clipboard API

```JS
copy() {
  navigator.ckipboard.writeText(this.sourceTarget.value)
}
```

Load the page in your browser and click the Copy button. Then switch back to your editor and paste. You should see the PIN `1234`.

### f. Stimulus Controller are Reusable

So far we've seen what happens when there's one instance of a controller on the page at a time.

It's not unusual to have multiple of a controller one the page simultaneously. For example. we might want to display a list of PINs, each with its own Copy button.

Our controller is reusable any time we want to provide a way to copy ab bit of text to the clipboard, all we need is markup on the page with the right annotations.

Let's go ahead and add another PIN to the page. Copy and paste the `<div>` so there are two identical PIN fields, then change the `value` attribute of the second:

```HTML
<div data-controller="clipboard">
  PIN: <input data-clipboard-target="sourec" type="text" value="3737" readonly>
  <button data-action="clipboard#copy">Copy to Clipboard</button>
</div>
```

Reload the page and confirm that both buttons work.

### g. Actions and Targets Can Go on ANy Kind of Element

Now let's add one more PIN field. This time we'll use a Copy link instead of a button:

```HTML
<div data-controller="clipboard">
  PIN: <input data-clipboard-target="source" type="text" value="3737" readonly>
  <a href="#" data-action="clipboard#copy">Copy to Clipboard</a>
</div>
```

Stimulus let us use any kind of element we want as long as it hash an appropriate `data-action` attribute.

Note that in this case, clicking the link will also cause the browser to follow the link's `href`. We can cancel this default behavior by calling `event.preventDefault()` in the action:

```JS
copy() {
  event.preventDefault();
  navigator.clipboard.writeText(this.sourceTarget.value)
}
```

Similar, our `source` target neef not be an `<input type="text">`. The controller only expects it be have a `value` property and a `select()` method. That means we can use a `<textarea>` instead:

```HTML
PIN: <textarea data-clipboard-target="source" readonly>3737</textarea>
```

### h. Wrap-Up and Next Steps

In this chapter we looked at a real-life example of wrapping a browser API in a Stimulus controller. We saw how multiple instances of the controller can appear on the page at once, and we explored how actions and targets keep your HTML and JavaScript loosely coupled.

Now let's see how small changes to the controller's design can lead us to a more robust implementation.

## <u>4. Designing For Resilience</u>

Although the clipboard API us [well-supported in current browsers](https://caniuse.com/clipboard), we might still expect to have a small number of people with older browser using our application.

We should also expect people to havce problem accessing our application from time to time. For example, intermitten network connectivity or CDN availablity could prevent some or all of our JavaScript from loading.

It's tempting to write off support for older browsers as not worth the effort, or to dismiss network isssues as temporary glitch that resolve themselves after a refresh. But aftem it's trivially easy to build features in a way that's gracefully resilient to these types of problems.

This resilient approach, commonly known as progressive enhancement, is the practice of delivering web interfaces such as the basic functionality is implemented in HTML and CSS, and tiered upgrades to that base experience are layered on top with CSS and JavaScript, progressively, when their underlying technologies are supported by the browser.

### a. Progressively Enhancing the PIN Field

Let's look at how we can progressively enhance our PIN field so that that Copy button is invisible unless it's supported by the browser. That way we can avoid showing someone a button that doesn't work.

We'll start by hiding the Copy button in CSS. Then we'll feature-test support for Clipboard API in our Stimulus controller. If the API is supported, we'll add a class name to the controller element to reveal the button.

We start off by adding `data-clipboard-supported-class="clipboard--supported"` to the `div` element that has the `data-controller` attribute:

```HTML
<div data-controller="clipboard" data-clipboard-supported-class="clipboard--supported">
```

Then add `class="slipboard-button"` to the button element:

```HTML
<button data-action="clipboard#copy" class="clipboard-button">Copy to Clipboard</button>
```

Then add the following styles to `public/main.css`

```CSS
.clipboard-button {
  display: none;
}

.clipboard--supported .clipboard-button {
  display: initial;
}
```

First we'll add the `data-clipboard-supported-class` attribute inside the controller as a static class:

```JS
static classes = ["supported"]
```

This will let us control the specific CSS class in the HTML, so our controller becomes even more easily adaptable to different CSS approaches. The specific class added like this can be access via `this.supportedClass`.

Now add a `connect()` method to the controller which will test to see if the clipboard API is supported and add a class name to the controler's element:

```JS
connect() {
  if ("clipboard" in navigator) {
    this.element.classList.add(this.supportedCLass);
  }
}
```

You can place this method anywhere in the controller's class body.

If you wish, disable JavaScript in your browser, reload the page, and notice the Copy button is no longer visible.

We have progressively enhance the PIN field: its Copy button's baseline state is hidden, becoming visible only when our JavaScript detects support for the clipboard API.

### b. Wrap-Up and Next Steps

In this chapter we gently modified our clipboard controller to be resilient against older browsers and degraded network conditions.

Next, we'll learn about how Stimulus controllers manage state.

## <u>5. Managing State</u>

Most contemporary frameworks encourage you to keep state in JavaScript at all time. They treat the DOM as a write-only renderin target, reconciled by client-side templates consuming JSOn from the server.

Stimulus takes a different approach. A stimulus application's state lives as attribute in the DOM; controller themselves are largely stateless. This approach makes it possible to work with HTML from anywhere - the initial document, and Ajax request, a Turbo visit, or event another JavaScript library - and have associated controller spring to life automatically without any explicit initialization step.

### a. Building a Slideshow

In the last chapter, we learned how a Stimulus controller can maintain simple state in the document by adding c class name to an element. But what we do when we need to store a value, not just a simple flag?

We'll investigate this question by building a slideshow controller which keeps its currently selected slide index in an attribute.

As usual, we'll begin with HTML

```HTML
<div data-controller="slideshow">
  <button data-action="slideshow#previous">←</button>
  <button data-action="slideshow#next">→</button>

  <div data-slideshow-target="slide">🐵</div>
  <div data-slideshow-target="slide">🙈</div>
  <div data-slideshow-target="slide">🙉</div>
  <div data-slideshow-target="slide">🙊</div>
</div>
```

Each `slide` target represents a single slide in the slideshow. Our controller will be responsible for making sure only one slide is visible at a time.

Let's draft our controller. Create a new file, `src/controllers/slideshow_controller.js`, as follows:

```JS
// src/controllers/slideshow_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["slide"]

  initialize() {
    this.index = 0
    this.showCurrentSlide()
  }

  next() {
    this.index++
    this.showCurrentSlide()
  }

  previous() {
    this.index--
    this.showCurrentSlide()
  }

  showCurrentSlide() {
    this.slideTargets.forEach((element, index) => {
      element.hidden = index !== this.index
    })
  }
}
```

Our controller defines a method, `showCurrentSlide()`, which loops over each slide target, toggling the [hidden attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/hidden) it its index matches.

We initialize the controller by showing the first slide, and the `next()` and `previous()` action methods advance and rewind the current slide.

> Lifecycle Callbacks Explained

What does the `initialize()` method do ? How it it different from the `connect()` method we've used before ?

These are Stimulus lifecycle callback methods, and they're useful for setting yp or tearing down associated state when your controller enters or leaves the document.

| Method       | Invoked by Stimulus                                 |
|--------------|-----------------------------------------------------|
| initialize() | Once, when the controller is first instantiated     |
| connect()    | Anytime the controller is connected to the DOM      |
| disconnect() | Anytime the controller is disconnected from the DOM |

Reload the page and confirm that the Next button advances to the next slide.

## b. Reading Initial State from the DOM

Notice how our controller tracks its state - the currently selected slide - in the `this.index` property.

Now say we'd like to start one of our slideshows with the second slide visible instead of the first. How can we encode the start index in our markup?

One way might be to load the initial index with an HTML `data` attribute. For example, we could add a `data-index` attribute to the controller's element:

```HTML
<div data-controller="slideshow" data-index="1>
```

Then, in our `initialize()` method, we could read that attribute, convert it to an integer, and pass it to `showCurrentSlide()`:

```JS
initialize() {
  this.index = Number(this.element.dataset.index)
  this.showCurrentSlide()
}
```

This might get the job done, but it's clunky, requires us to make a decision about what to name the attribute, and doesn't help us if we want to access the index again or increment it and persist the result in the DOM.

### c. Using Values

Stimulus controllers support typed value property which automaticaly map to data attributes. When we add a value definition to the top of our controller class:

```JS
static values = { index: Number }
```

Stimulus will create a `this.indexValue` controller property associated with a `data-slideshow-index-value` attribute, and handle the numeric conversion for us.

Let's see that in action. And the associated data attribute to our HTML:

```HTML
<div data-controller="slideshow" data-slideshow-index-value="1">
```

Than add a `static values` definition to the controller and change the `initialize()` method to log `this.indexValue`:

```JS
export default class extends Controller {
  static values = { index: Number }

  initialize() {
    console.log(this.indexValue)
    console.log(typeof this.indexValue)
  }
}
```

Reload the page and verify that the console show `1` and `Number`.

> What's with that `static values` line ?

Similiar to targets, you define values in a Stimulus controller ny describing them in a static object property called `values`. In this case, we've defined a single numeric value called `index`. You can read more about value definitions in the [reference documentation](https://stimulus.hotwired.dev/reference/values).

Now let's update `initialize()` and the other methods in the controller to use `this.indexValue` instead of `this.index`. Here how the controller should look when we're done.

```JS
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["slide"]
  static values = { index: Number }

  initialize() {
    this.showCurrentSlide()
  }

  next() {
    this.indexValue ++
    this.showCurrentSlide()
  }

  previous() {
    this.indexValue --
    this.showCurrentSlide()
  }

  showCurrentSLide() {
    this.slideTargets.forEach((element, index) => {
      element.hidden = index !== this.indexValue
    })
  }
}
```

Reload the page and use the web inspector to confirm the controller element's `data-slideshow-index-value` attribute changes as you move from one slide to the next.

### d. Change Callbacks

Our revised controller improves on the original version, but the repeated calls to `this.showCurrentSlide()` standout. We have to manually update the state of the document when the controller initializes after every place where we update `this.indexValue`.

We can define a Stimulus value change callback to clean up the repetition and specify how the controller should respond whenever the index value changes.

First, remove the `initialize()` method and definde a new method, `indexValueChanged()`. Then remove the calls to `this.showCurrentSlide()` from `next()` and `previous()`:

```JS
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["slide"]
  static values = { index: Number }

  next() {
    this.indexValue++
  }

  previous() {
    this.indexValue++
  }

  indexValueChanged() {
    this.showCurrentSlide()
  }

  showCurrentSlide() {
    this.slideTargets.forEach((element, index) => {
      element.hidden = index !== this.indexValue
    })
  }
}
```

Reload the page and confirm ths slideshow behavior is the same.

Stimulus calls the `indexValueChanged()` method at initialization and in response to any change to the `data-slideshow-index-value` attribute. You can even fiddle with the attribute in the web inspector and the controller will change sliudes in response. Go ahead-try it out!

### e. Settings Defaults

It's also possible to set a default values as part of the static definition. This is done like so:

```JS
static values = { index: { type: Number, default: 2 } }
```

That would start the index at 2, if no `data-slideshow-index-value` attribute was defined on the controller element. If you had other values, you can mix and match what needs a default and what doesn't:

```JS
static values = { index: { type: Number, default: 2 }, effect: { type: String, default: "kenburns" } }
```

### f. Wrap-Up and Next Steps

In this chapter we've seen how to use the values to load and persist the current index of a slideshow controller.

From a usability perspective, our controller is incomplete. The Previous button appears to do nothing when you are looking at the first slide. Internally, `indexValue` decrement from `0` to `-1`. Could we make the value wrap around to the last slide index instead? (There's a similar problem with the Next button).

Next we’ll look at how to keep track of external resources, such as timers and HTTP requests, in Stimulus controllers.

## <u>6. Working With External Resources</u>

In the last chapter we learned how to load and persist a controller's internal state using values.

Sometimes our controller need to track the state of external resources, where by external we mean anything that isn't in the DOM or a part of Stimulus. For example, we may need to issue an HTTP request and respond as the request's state changes. Or we may want to start a timer and then stop it when the controller is no longer connected. In thif chapter we'll how to do both of those things.

### a. Asynchronously Loading HTML

Let's learn how to populate parts of a page asynchronously by loading and inserting remote fragments of HTML. We use this technique in Basecamp to keep our initial page loads fast, and to keep our views free of user-specific content so they can be cached more effectively.

We'll build a general-purpose content loader controller which populates its element with HTML fetched from the server. Then we'll use it to load a list of unread messages like you'd see in an email inbox.

Begin by sketching the inbox in `public/index.html`:

```HTML
<div data-controller="content-loader"
     data-content-loader-url-value="/messages.html">
</div>
```

Then create a new `public/messages.html` file with some HTML for our message list:

```HTML
<ol>
  <li>New Message: Stimulus Launch Party</li>
  <li>Overdue: Finish Stimulus 1.0</li>
</ol>
```

(In a real application you'd generate this HTML dynamically on the server, but for demonstration purpose we;ll just use a static file.)

Now we can implement our controller:

```JS
// src/controllers/content_loader_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = ["url"]

  connect() {
    this.load()
  }

  load() {
    fetch(this.urlValue)
      .then(response => response.text())
      .then(html => this.element.innerHTML = html)
  }
}
```

When the controller connects, we kick off a [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) request to the URL specified in the element's `data-content-loader-url-value` attribute. Then we load the returned HTML by assigning it to our element's `innerHTML` property.

Open the network tab in your browser's developer console and reload the page. You'll see a request representing the initial page load, followed by our controller's subsequent request to `messages.html`.

### b. Refreshing Automatically With a Timer

Let's improve our controller by changing it to periodically refresh the inbox so it;s always up-to-date.

We'll use the `data-content-loader-refresh-interval-value` attribute to specify how often the controller should reload its contents, in miliseconds:

```HTML
<div data-controller="content-loader"
     data-content-loader-url-value="/messages.html"
     data-content-loader-refresh-interval-value="500">
</div>
```

Now we can update the controller to check for the interval and, if present, start a refresh timer.

Add a `static values` definition to the controller, and define a new method `startRefreshing()`:

```JS
export default class extends Controller {
  static values = { url: String, refreshInterval: Number }

  startRefreshing() {
    setInterval(() => {
      this.load()
    }, this.refreshIntervalValue)
  }
}
```

Then update the `connect()` method to call `startRefreshing()` if an interval value is present.

```JS
conenct() {
  this.load()

  if (this.hasRefreshingIntervalvalue) {
    this.startRefreshing()
  }
}
```

Reload the page and observe a new request once every five seconds in the developer console. Then make a change to `public/messages.html` and wait for it to appear in the inbox.

### c. Releasing Tracked Resources

We start our timer when the controller connects, but we never stop it. That means if our controller's element were to disappear, the controller would continue to issue HTTP trquests in the background.

We can fix this issue by modifying our `startRefreshing()` method to keep a reference to the timer:

```JS
startRefreshing() {
  this.refreshTimer = setInterval(() => {
    this.load()
  }, this.refreshInterValue)
}
```

Then we can add a corresponding `stopRefreshing()` method below to cancel the timer:

```JS
stopRefreshing() {
  if (this.refreshTimer) {
    clearInterval(this.refreshTimer)
  }
}
```

Finally, to instruct Stimulus to cancel the timer when the controller disconnect, we'll `disconnect()` method:

```JS
disconnect() {
  this.stopRefreshing()
}
```

Now we can be sure a content loader controller will only issue requests when it's connected to the DOM.

Let's take a look at our final controller class:

```JS
// src/controllers/content_loader_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = { url: String, refreshInterval: Number }

  connect() {
    this.load()

    if (this.hasRefreshIntervalValue) {
      this.startRefreshing()
    }
  }

  disconnect() {
    this.stopRefreshing()
  }

  load() {
    fetch(this.urlValue)
      .then(response => response.text())
      .then(html => this.element.innerHTML = html)
  }

  startRefreshing() {
    this.refreshTimer = setInterval(() => {
      this.load()
    }, this.refreshIntervalValue)
  }

  stopRefreshing() {
    if (this.refreshTimer) {
      clearInterval(this.refreshTimer)
    }
  }
}
```

### d. Using action parameters

If we wanted to make the loader work with multiple different sources, we could do it using action parameters. Take this HTML:

```HTML
<div data-controller="content-loader">
  <a href="#" data-controller-loader-url-params="/messages.html" data-action="content-loader#load">
    Messages
  </a>
  <a href="#" data-controller-loader-url-params="/contents.html" data-action="content-loader#load">
    Contents
  </a>
</div>
```

We could even destruct the params to just get the URL parameter:

```JS
load({ params: { ulr }}) {
  fetch(url)
    .then(response => response.text())
    .then(html => this.element.innterHTML = html)
}
```

### e. Wrap-Up and Next Steps

In this chaptert we've seen how to acquire and release external resources using Stimulus lifecycle callbacks.

Next we'll see how to install and configure Stimulus in your own application.

## <u>7. Installing Stimulus in Your Application</u>

To install Stimulus in your application, add the [@hotwired/stimulus npm package](https://www.npmjs.com/package/@hotwired/stimulus) to your JavaScript bundle. Or, import [stimulus.js](https://unpkg.com/@hotwired/stimulus@3.2.1/dist/stimulus.js) in a `<script type="module">` tag.

### a. Using Stimulus for Rails

If you're using [Stimulus for Rails](https://github.com/hotwired/stimulus-rails/) together with an [import map](https://github.com/rails/importmap-rails), the integration will automatically load all controller files from `app/javascript/controllers`.
