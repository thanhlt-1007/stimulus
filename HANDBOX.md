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

## <u>1. Introduction</u>

## <u>2. Hello, Stimulus</u>

## <u>3. Building Something Real</u>

## <u>4. Designing For Resilience</u>

## <u>5. Managing State</u>

## <u>6. Working With External Resources</u>

## <u>7. Installing Stimulus in Your Application</u>
