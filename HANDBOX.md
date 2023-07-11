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

```js
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

```js
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

```js
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

```js
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

```js
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

```js
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

Then add `data-controller="clipboard"` to the outer `<div>`. Any time this attributes appears on an element, Stimulus will connect an instance of our controller:

```HTML
<div data-controller="clipboard">
```

## <u>4. Designing For Resilience</u>

## <u>5. Managing State</u>

## <u>6. Working With External Resources</u>

## <u>7. Installing Stimulus in Your Application</u>
