# REFERENCE

## <u>1. Controllers</u>

A controller is the basic organizational unit of a Stimulus application.

```JS
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {

}
```

Controller are instances of JavaScript classes that you define in your application. Each controller class inherits from the `Controller` base class exported by the `@hotwired/stimulus` module.

### a. Properties

Every controller belongs to a Stimulus `Application` instance and is associated with an HTML element. Within a controller class, you can access the controller's:

- application, via the `this.application` property

- HTML element, via the `this.element` property

- identifier, via the `this.identifier` property

### b. Modules

Define your controller classes in JavaScript modules, one per file.Export each controller class as the module's default object, as in the example above.

Place these modules in the `controllers/` directory. Nam the files `[identifier]_controller.js` where `[identifier]` corresponds to each controller's identifier.

### c. Identifiers

An identifier is the name you use to reference a controller class in HTML.

When you add a `data-controller` attribute to an element, Stimulus reads the identifier from the attribute's value and creates a new instance of the corresponding controller class.

For example, this element has a controller which is an instance of the class defined in `controllers/reference_controller.js`

```HTML
<div data-controller="reference"></div>
```

The following is an example of how Stimulus will generate identifiers for controllers in its require context:

| If your controller file is named... | Its identifier will be... |
| ----------------------------------- | ------------------------- |
| clipboard_controller.js             | clipboard                 |
| date_picker_controller.js           | date-picker               |
| users/list_item_controller.js       | users--list-item          |
| local-time-controller.js            | local-time                |

### d. Scopes

When Stimulus connects a controller to an element, that element and all of its children make up the controller's scope.

For example, the `<div>` and `<h1>` below are part of the controller's scope, but the surrounding `<main>` element is not.

```HTML
<main>
  <div data-controller="reference">
    <h1>Reference</h1>
  </div>
</main>
```

### e. Nested Scopes

When nested, each controller is only aware of its own scope excluding the scope of any controllers nested within.

For example, the `#parent` controller below is only aware of the `item` targets directly within its scope, but not any targets of the `#child` controller.

```HTML
<ul id="parent" data-controller="list">
  <li data-list-target="item">One</li>
  <li data-list-target="item">Two</li>
  <li>
    <ul id="child" data-controller="list">
      <li data-list-target="item">I am</li>
      <li data-list-target="item">a nested list</li>
    </ul>
  </li>
</ul>
```

### f. Multiple Controllers

The `data-controller` attribute's value is a space-separated list of identifiers:

```HTML
<div data-controller="clipboard list-item"></div>
```

It's common for any given element on the page to have many controllers. In the example above, the `<div>` has two connected controllers, `clipboard` and `list-item`

Similary, it's common for multiple elements on the page to reference the same controller class:

```HTML
<ul>
  <li data-controller="list-item">One</li>
  <li data-controller="list-item">Two</li>
  <li data-controller="list-item">Three</li>
</ul>
```

Here, each `<li>` has its own instance of the `list-item` controller.

### g. Naming Conventions

Always use camelCase for method and property names in a controller class.

When an identifier is composed of more than one word, write the words in kebab-case (i.e,. by using dashes: `date-picker`, `list-item`).

In filenames, separate multiple words using either underscores or dashes (snake_case or kebab-case: `controllers/date_picker_controller.js`, `controllers/list-item-controller.js`).

### h. Registration

If you use Stimulus for Rails with and import map or Webpack together with the `@hotwired/stimulus-weboack-helpers` package, your application will automatically load and register controller classes following the conventions above.

If not, your application must manually load and register each controller class.

#### i. Registering Controllers Manually

To manually register a controller class with an identifier, first import the class, then call the `Application#register` method on your application object:

```JS
import ReferenceController from "./controllers/reference_controller"

application.register("reference", ReferenceController)
```

You can also register a controller class inline instead of importing it from a module:

```JS
import { Controller } from "@hotwired/stimulus"

application.register("reference", class extends Controler {

})
```

#### ii. Preventing Registration Based On Environmental Factors

If you only want a controller registered and loaded if certain environmental factors are met - such a given user agent - you can override the static `shouldLoad` method:

```JS
class UnloadableController extends ApplicationController {
  static get shouldLoad() {
    return false
  }
}

// This controller will not be loaded
application.register("unloadable", UnloadableController)
```

#### iii. Trigger Behaviour When A Controller Is Registered

If you want to trigger some behaviour once a controller has been registered you can add a static `afterLoad` method:

```JS
class SpinnerButton extends Controller {
  static afterLoad(identifier, application) {
    // use the application instance to read the configured 'data-controller' attribute
    const { controllerAttribute } = application.schema

    // update any legacy buttons with the controller's registered identifier
    const updateLegacySpinners = () => {
      document.querySelector(".legacy-spinner-button").forEach((element) => {
        element.setAttribute(controllerAttribute, identifier)
      })
    }

    // called as soon as registered so DOM may not have loaded yet
    if () {
      document.addEventListener("DOMContentLoaded", updateLegacySpinners)
    } else {
      updateLegacySpinners()
    }
  }
}

// This controller will update any legacy spinner buttons to use the controller
application.register("spinner", SpinnerButton)
```

The `afterLoad` method will get called as soon as the controller has been registered, even if no controlled elements exist in the DOM. It gets called with the `identifier` that was used when registering the controller and the Stimulus application instance.

#### iv. Cross-Controller Coordination With Events

If you need controllers to communicate with each other, you should use events. The `Controller` class has a convenience method called `dispatch` that makes this easier. It takes an `eventName` as the first argument, which is then automatically prefixed with the name of the controller seperated by a colon. The payload is held in `detail`. It work like this:

```JS
class ClipboardController extends Controller {
  static targets = ["source"]

  copy() {
    this.dispatch("copy", { detail: { content: this.sourceTarget.value } })
    navigator.clipboard.writeText(this.sourceTarget.value)
  }
}
```

And this event can then be routed to an action on another controller

```HTML
<div data-controller="clipboard effects" data-action="clipboard:copy->effects#flash">
  PIN: <input data-clipboard-target="source" type="text" value="1234" readonly>
  <button data-action="clipboard#copy">Copy ti Clipboard</button>
</div>
```

So when `Clipboard#copy` action is invoked, the `Effect#flash` action is be too:

```JS
class EffectController extends Controller {
  flash({ detail: { content }}) {
    console.log(content) // 1234
  }
}
```

`dispatch` accepts addtional options as the second parameter as follow

| option     | default         | notes                                                                                                                                                                                     |
|------------|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| detail     | {} empty object | See [CustomEvent.detail](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent/detail)                                                                                             |
| target     | this.element    | See [Event.target](https://developer.mozilla.org/en-US/docs/Web/API/Event/target)                                                                                                         |
| prefix     | this.identifier | If the prefix is falsy (e.g: `null` or `false`), only the `eventName` will be used. If you provide a string value the `eventName` will be prepended with the provided string and a colon. |
| bubbles    | true            | See [Event.bubbles](https://developer.mozilla.org/en-US/docs/Web/API/Event/bubbles)                                                                                                       |
| cancelable | true            | See [Event.cancelable](https://developer.mozilla.org/en-US/docs/Web/API/Event/cancelable)                                                                                                 |

`dispatch` will return the generated [CustomEvent](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent), you can use this to provide a way for the event to be cancelled by any other listeners as follows:

```JS
class ClipboardController extends Controller {
  static targets = ["source"]

  copy() {
    const event = this.dispatch("copy", { cancellable: true })
    if (event.defaultPrevented) return

    navigator.clipboard.writeText(this.sourceTarget.value)
  }
}

class Effects extends Controllers {
  flash(event) {
    // this will prevent the default behaviour as determined by the dispatch event
    event.preventDefault()
  }
}
```

#### v. Directly Invoking Other Controllers

If for some reason it is not possible to use events to communicate between controllers, you can reach a controller instance via the `getControllerForElementAndIdentifier` method from the application. This should only be used if you have a unique problem that cannot be solved through the more general way of using events, but if you must, this is how:

```JS
class MyController extends Controller {
  static targets = ["other"]

  copy() {
    const otherController = this.application,getControllerFromElementAndIdentifier(this.otherTarget, "other")
    otherController.otherMethod()
  }
}
```

## <u>2. Lifecycle Callbacks</u>

Special methods called lifecycle callbacks allow you to respond whenever a controller or certain targets connects to and disconnects from the document.

```JS
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  connect() {
    // ..
  }
}
```

### a. Methods

You may define any of the following methods in your controller"

| Method                                    | Invoked by Stimulus...                              |
|-------------------------------------------|-----------------------------------------------------|
| initialize()                              | Once, when the controller is first instantiated     |
| [name]TargetConnected(target: Element)    | Anytime a target is connected to the DOM            |
| connect()                                 | Anytime the controller is connected to the DOM      |
| [name]TargetDisconnected(target: Element) | Anytime a target is disconnected from the DOM       |
| disconnect()                              | Anytime the controller is disconnected from the DOM |

### b. Connection

A controller is connected to the document when both of the following conditions are true:

- its elements is present in the document (i.e., a descendant of `document.documentElement`, the `<html>` element)

- its identifier is present in the element's `data-controller` attribute

When a controller becomes connected, Stimulus calls its `connect()` method.

#### i. Targets

A target is connected to the document when both of the following conditions are true:

- its element is present in the document as a descendant of its corresponding controller's element

- its identifier is present in the element's `data-{identifier}-target` attribute

When a target becomes disconnected, Stimulus calles its controller's `[name]TargetDisconnected()` method, passing the target element as a parameter. The `[name]TargetDisconnected()` lifecycle callbacks will fire before the controller's `disconnect()` callback.

### c. Reconnection

A disconnected controller may become connected again at a later time

When this happens, such as after removing the controller's element from the document and then re-attaching it, Stimulus will reuse the element's previous controller instance, calling its `connect()` method multiple times.

Similarly, a disconnected target may be connected again at a later time. Stimulus will invoke its controller's `[name]TargetConnected()` method multiple times.

### d. Order and Timing

Stimulus watches the page for changes asynchronously using the [DOM MutationObserve API](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver).

This means that Stimulus calls your controller's lifecycle methods asynchronously after changes are made to the document, in the next [microtask](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/) following each change.

Lifecycle methods still run in the order they occur, so two calls to a controller's `connect` method will always be separated by one call to `disconnect()`. Similarly, two calls to a controller's `[name]TargetConnected()` for a given target will ways be separated by one call to `[name]TargetDisconnected()` for that same target.

## <u>3. Actions</u>

Actions are how you handle DOM events in your controllers.

```HTML
<div data-controller="gallery">
  <button data-action="click->gallery#next">...</button>
</div>
```

```JS
// controllers/gallery_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  next(event) {
    // ...
  }
}
```

An action is a connection between:

- a controller method

- the controller's element

- a DOM event listener

### a. Descriptors

The `data-action` value `click->gallery#next` is called an action descriptor. In this descriptor:

- `click` is the name of the DOM event to listen for

- `gallery` is the controller identifier

- `next` is the name of the method to invoke

#### i. Event Shorthand

Stimulus lets you shorten the action descriptors for some common element/event pairs, such as the button/click pair abovem by ommiting the event name:

```HTML
<button data-action="gallery#next">...</button>
```

The full set of these shorthand pairs is as follows:

| Element           | Default Event |
|-------------------|---------------|
| a                 | click         |
| button            | click         |
| details           | click         |
| form              | click         |
| input             | input         |
| input type=submit | click         |
| select            | change        |
| textarea          | input         |

### b. KeyboardEvent Filter

There may be cases where [KeyboardEvent](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent) Actions should only call the Controller method when certain keystrokes are used.

You can install an event listener that responds only to the `Escape` by adding `.esc` to the event name of the action descriptor, as in the following example.

```JS
<div data-controller="model"
  data-action="keydown.esc->modal#close" tabindex="0">
</div>
```

This will only work if the event being fierd is a keyboard event.

The correspondence between these filter and keys is shown below.

| Filter | Key Name   |
|--------|------------|
| enter  | Enter      |
| tab    | Tab        |
| esc    | Escape     |
| space  | ""         |
| up     | ArrowUp    |
| down   | ArrowDown  |
| left   | ArrowLeft  |
| right  | ArrowRight |
| home   | Home       |
| end    | End        |
| [a-z]  | [a-z]      |
| [0-9]  | [0-9]      |

If you need to support other keys, you can customize the modifiers using a custom schema.

```JS
import { Application, defaultSchema } from "@hotwired/stimulus"

const customScheme = {
  ...defaultSchema,
  keyMappings: { ...defaultSchema.keyMappings, at: "@" },
}

const app = Application.start(document.documentElement, customSchema)
```

If you want to subscribe to a compound filter using a modifier key, you can write it like `ctrl + a`.

```HTML
<div data-action="keydown.ctrl+a->listbox#selectAll" role="option" tabindex="0">...</div>
```

The list of supported modifier keys is shown below.

| Modifier | Notes                |
|----------|----------------------|
| alt      | options for MacOS    |
| ctrl     |                      |
| meta     | Command key on MacOS |
| shift    |                      |

#### i. Global Events

Sometimes a controller needs to listen for events dispatched on the global `window` or `document` objects.

You can append `@window` or `@document` to the event name (along with any filter modifer) in an action descriptor to install the event listener on `window` or `document`, respectively, as in the following example:

```HTML
<div data-controller="gallery"
     data-action="resize@window->gallery#resize">
</div>
```

#### ii. Options

You can append one or more action options to an action descriptor if you need to specify [DOM event listener options](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#parameters).

```HTML
<div data-controller="gallery"
     data-action="scroll->gallery#layout:!passive">
  <img data-action="click->gallery#open:capture">
</div>
```

Stimulus supports the following action options:

| Action options | DOM event listener option |
|----------------|---------------------------|
| :capture       | { capture: true }         |
| :once          | { once: true }            |
| :passive       | { passive: true }         |
| :!passive      | { passive: false }        |

On top of that, Stimulus also supports the following action options which are not natively supported by the DOM event listener options:

| Custom action option | Description                                                          |
|----------------------|----------------------------------------------------------------------|
| :stop                | calls `.stopProgagation()` on the event before invoking the method   |
| :prevent             | calls `.preventDefault()` on the event before invoking the method    |
| :self                | only invokes the method if the event was fired by the element itself |

You can register your own action options with the `Application.registerActionOption` method.

For example, consider that a `<details>` element will dispatch a toggle event whenever it's toggled. A custom `:open` action option would help to route events whenever the element is toggled open:

```JS
import { Application } from "@hotwired/stimulus"

const application = Application.start()

application.registerActionOption("open", ({ event }) => {
  if (event.type == "toggle") {
    return event.target.open == true
  } else {
    return true
  }
})
```

Similarly, a custom `:!open` action option could route events whenever the element is toffled closed. Declaring the action descriptor option with a `!` prefix will yield a `value` argument set to `false` in the callback:

```JS
import { Application } from "@hotwired/stimulus"

const application = Application.start()

application.registerActionOption("open", ({ event, value}) => {
  if (event.type == "toggle") {
    return event.target.open == value
  } else {
    return true
  }
})
```

In order to prevent the event from being routed to the controller action, the `registerActionOption` callback function must return `false`. Otherwise, to routes the event to the controller action, return `true`.

The callback accepts a single object argument with the following keys:

| Name    | Description                                                                                |
|---------|--------------------------------------------------------------------------------------------|
| name    | String: The option's name (`"open"` in the example above)                                  |
| value   | Boolean: The value of the option(`:open` would yield `true`, `:!open` would yield `false`) |
| event   | `Event`: The event instance                                                                |
| element | `Element`: The element where the action descriptor is declared                             |

### c. Event Objects

An action method is the method in a controller which serves as an action's event listener.

The first argument to an action method is the DOM event object. You may want access to the event for a number of reasons, including:

- to read the key code from a keyboard event

- to read the coordinates of a mouse event.

- to read data from an input event.

- to read params from the action submitter element

- to prevent the browser's default behaviour for an event

- to find out which element dispatched an event before it bubbled up to this action.

The following basic properties are common to all events:

| Event Property      | Value                                                                                                                                |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| event.type          | The name of the event (e.g. `"click"`)                                                                                               |
| event.target        | The target that dispatched the event (i.e. the innermost element that was clicked)                                                   |
| event.currentTarget | The target on which the event listener is installed (either the element with the `data-action` attribute, or `document` or `window`) |
| event.params        | The action params passed by the action submitter element                                                                             |

The following evet methods give you more control over how events are handled:

| Event Method            | Result                                                                            |
|-------------------------|-----------------------------------------------------------------------------------|
| event.preventDefault()  | Cancels the event's defaulr behavior (e.g. following a link or submitting a form) |
| event.stopPropagation() | Stops the event before it bubbles up to other listeners on parent elements        |

### d. Multiple Actions

The `data-action` attribute's value is a space-separated list of action descriptors.

It's common for any given element to have many actions. For example, the following input element calls a `field` controller's `highlight()` method when it gains focus, and a `search` controller's `update()` method every time the element's value changes:

```HTML
<input type="text" data-action="focus->field#highlight input->search#update">
```

When an element has more than one action for the same event, Stimulus invokes the actions from left to right in the order that their descriptors appear.

The action chain can be stopped at any point by calling `Event#stopImmediatePropagation()` within an action. Any addtional actions to the right will be ignored:

```JS
highlight: function(event) {
  event.stopImmediatePropagation()
  // ..
}
```

### e. Naming Conventions

Always use camelCase to specify action names, since they map directly to methods on your controller.

Avoid action names that simply repeat the event's name, such as `click`, `onClick`, or `handleClick`:

```HTML
<button data-action="profile#click">Don't</button>
```

Instead, name your action methods based on what will happen when they're called:

```HTML
<button data-action="click->profile#showDialog">Do</button>
```

This will help you reason about the behavior of a block of HTML without having to look at the controller source.

### f. Action Parameters

Actions can have parameters that are be passed from the submitter element. They follow the format of `data-[identifier]-[param-name]-param`. Parameters must be specified on the same element as the action they intend to be passed to is declared.

All parameters are automatically typecast to either a `Number`, `String`, `Object`, or `Boolean`, inferred by their value:

| Data attribute                                   | Param              | Type    |
|--------------------------------------------------|--------------------|---------|
| `data-item-id-params="12345"`                    | `12345`            | Number  |
| `data-item-url-param="/votes"`                   | `"/votes"`         | String  |
| `data-item-payload-param='{"value": "1234567"}'` | `{value: 1234567}` | Object  |
| `data-item-active-param="true"`                  | `true`             | Boolean |

Consider this setup:

```HTML
<div data-controller="item spinner">
  <button data-action="item#upbote spinner#start"
    data-item-id-param="12345"
    data-item-url-param="/votes"
    data-item-payload-param='{"value": "1234567"}'
    data-item-active-param="true">...</button>
  </button>
</div>
```

It will call both `ItemController#upvote` and `SpinnerController#start`, but only the former will have any parameters passed to it:

```JS
// ItemController
upvote(event) {
  // { id: 12345, url: "/votes", active: true, payload: { value: 1234567 } }
  console.log(event.params)
}

// SpinnerController
start(event) {
  // {}
  console.log(event.params)
}
```

If we don't need anything else from the event, we can destruct the params

```JS
upvote({ params }) {
  // { id: 12345, url: "/votes", active: true, payload: { value: 1234567 } }
  console.log(params)
}
```

Or destruct only the params we need, in case multiple actions on the same controller share the same submitter element:

```JS
update({ params: { id, url} }) {
  console.log(id) // 12345
  console.log(url) // "/votes"
}
```

## <u>4. Targets</u>

Targets let you reference important elements by name

```HTML
<div data-controller="search">
  <input type="text" data-search-target="query">
  <div data-search-target="errorMessage"></div>
  <div data-search-target="results"></div>
</div>
```

### a. Attributes and Names

The `data-search-target` attribute is called a target attribute, and its value is a space-separated list of target names which you can use to refer to the element in the `search` controller.

```HTML
<div data-controller="search">
  <div data-search-target="results"></div>
</div>
```

### b. Definitions

Define target names in your controller class using the `static targets` array:

```JS
// controllers/search_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["query", "errorMessage", "results"]
  // ...
}
```

### c. Properties

For each target name defined in the `static targets` array, Stimulus adds the following properties to your controller, where `[name]` corresponds to the target's name:

| Kind        | Name                   | Value                                                            |
|-------------|------------------------|------------------------------------------------------------------|
| Singular    | `this.[name]Target`    | The first matching target in scope                               |
| Plural      | `this.[name]Targets`   | An array of all matching targets in scope                        |
| Existential | `this.has[name]Target` | A boolean indicating whether there is a matching target in scope |

Note: Accessing the singular target property will throw an error when there is no matching element.

### d. Shared Targets

Elements can have more than one target attribute, and it's common for targets to be shared by multiple controllers.

```HTML
<form data-controller="search checkbox">
  <input type="checkbox" data-search-target="projects" data-checkbox-target="input">
  <input type="checkbox" data-search-target="messages" data-checkbox-target="input">
  ...
</form>
```

In the example above, the checkboxes are accessible inside the `search` controllers as `this.projectsTarget` and `this.messageTarget`, respectively.

Inside the `checkbox` controller, `this.inputTargets` returns an array with both checkboxes.

### e. Optional Targets

If your controller needs to work with a target which may or may not be present, condition your code based on the value of the existential target property:

```JS
if (this.hasResultsTarget) {
  this.resultsTarget.innerHTML = "..."
}
```

### f. Connected and Disconnected Callbacks

Target element callbacks let you respond whenever a target element is added or removed within the controller's element.

Define a method `[name]TargetConnected` or `[name]TargetDisconnected` in the controller. where `[name]` is the name of the target you want to observe for additions or removals. The method receives the element as the first argument.

Stimulus invokes each element callback any time its target elements are added or removed after `connect()` and before `disconnect()` lifecycle hooks.

```JS
export default class extends Controller {
  static targets = ["item"]

  itemTargetConnected(element) {
    this.sortElements(this.itemTargets)
  }

  itemTargetDisconnected(element) {
    this.sortElements(this.itemTargets)
  }

  // private

  sortElements(itemTargets) { /*...*/ }
}
```

Note During the execution of `[name]TargetConnected` and `[name]TargetDisconnected` callbacks, the `MutationObserver` instances behind the scenes are paused. This means that if a callback add or removes a target with a matching name, the corresponding callback will not be invoked again.

### g. Naming Conventions

Always use camelCase to specify target names, since they map directly to properties on your controller:

```HTML
<span data-search-target="camelCase" />
<span data-search-target="do-not-do-this" />
```

```JS
export default class extends Controller {
  static targets = ["camelCase"]
}
```

## <u>5. Outlets</u>

Outlets let you reference Stimulus controller instances and their controller element from within another Stimulus Controller by using CSS selectors.

The use of Outlets helps with cross-controller communication and coordination as an alternative to dispatching custom events on controller elements.

They are conceptually similar to [Stimulus Targets](https://stimulus.hotwired.dev/reference/targets) but with the difference that they reference a Stimulus controller instance plus its associated controller element.

```HTML
<div data-controller="search" data-search-result-outlet=".result">
  ...
<div>

...

<div id="results">
  <div class="result" data-controller="results">...</div>
  <div class="result" data-controller="results">...</div>
</div>
```

While a target is a specifically marked element within the scope of its own controller element, an outlet can be located anywhere on the page and doesn't necessarily have to be within the controller scope.

### a. Attributes and Names

The `data-search-result-outlet` attribute is called an outlet attribute, and its value is a [CSS selector](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors) which you can use to refer to other controller elements which should be available as outlets on the host controller.

```JS
data-[identifier]-[outlet]-outlet="[selector]"
```

```HTML
<div data-controller="search" data-search-result-outlet=".result"></div>
```

### b. Definitions

Define controller identifiers in your controller class using the `static outlets` array. This array declares which other controller identifiers can be used as outlets on this controller:

```JS
// search_controller.js

export default class extends Controller {
  static outlets = ["result"]

  connect() {
    this.resultOutlets.forEach(result => ...)
  }
}
```

### c. Properties

For each outlet defined in the `static outlets` array, Stimulus adds five properties to your controller, where `[name]` corresponds to the outlet's controller identifier:

| Kind        | Property name          | Return Type         | Effect                                                                                                   |
|-------------|------------------------|---------------------|----------------------------------------------------------------------------------------------------------|
| Existential | `has[Name]Outlet`      | `Boolean`           | Tests for presence of a `[name]` outlet                                                                  |
| Singular    | `[name]Outlet`         | `Controller`        | Returns the `Controller` instance of the first `[name]` outlet or throws an exception if none is present |
| Plural      | `[name]Outlets`        | `Array<Controller>` | Returns the `Controller` instances of all `[name]` outlets                                               |
| Singular    | `[name]OutletElement`  | `Element`           | Returns the Controller `Element` of the first `[name]` outlet or throws an exception if none is present  |
| Plural      | `[name]OutletElements` | `Array<Element>`    | Returns the Controller `Element`'s of all `[name]` outlets                                               |

### d. Accessing Controllers and Elements

Since you get back a `Controller` instance from the `[name]Outlet` and `[name]Outlets` properties you are also able to access the Values, Classes, Targets and all of the other properties and functions that controller instance defines:

```JS
this.resultOutlet.idValue
this.resultOutlet.imageTarget
this.resultOutlet.activeClasses
```

You are also able to invoke any function the outlet controller may define:

```JS
// result_controller.js

export default class extends Controller {
  markAsSelected(event) {
    // ...
  }
}

// search_controller.js

export default class extends Controller {
  static outlets = ["result"]

  selectAll(event) {
    this.resultOutlets.forEach(result => result.markAsSelected(event))
  }
}
```

Similarly with the Outlet Element, it allows you to call any function or property on [Element](https://developer.mozilla.org/en-US/docs/Web/API/Element):

```JS
this.resultOutletElement.dataset.value
this.resultOutletElement.getAttribute("id")
this.resultOutletElements.map(result => result.hasAttribute("selected"))
```

### e. Outlet Callbacks

Outlet callbacks are specially named functions called by Stimulus to let you respond to whenever an outlet is added or removed from the page.

To observe outlet changes, define a function named `[name]OutletConnected()` or `[name]OutletDisconnected()`.

```JS
// search_controller.js

export default class extends Controller {
  static outlets = ["result"]

  resultOutletConnected(outlet, element) {
    // ...
  }

  resultOutletDisconnected(outlet, element) {
    // ...
  }
}
```

#### i. Outlets are Assumed to be Present

When you access an Outlet property in a Controller, you assert that at least one corresponding Outlet is present. If the declaration is missing and no matching outlet is found Stimulus will throw an exception:

```JS
Missing outlet element "result" for "search" controller
```

#### ii. Optional outlets

If an outlet is optional or you want to assert that at least Outlet is present, you must first check the presence of the Outlet using the existential property:

```JS
if (this.hasResultOutlet) {
  this.resultOutlet.safelyCallSomethingOnTheOutlet()
}
```

#### iii. Referencing Non-Controller Elements

Stimulus will throw an exception if you try to declare an element as an outlet which doesn't have a corresponding `data-controller` and identifier on it:

```HTML
<div data-controller="search" data-search-result-outlet="#result"></div>

<div id="result">></div>
```

Would result in:

```JS
Missing "data-controller=result" attribute on outlet element for "search" controller
```

## <u>6. Values</u>

You can read an write [HTML data attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/data-*) on controller elements as typed values using special controller properties.

```HTML
<div data-controller="loader"
     data-loader-url-value="/messages">
</div>
```

```JS
// controllers/loader_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = {
    url: String
  }

  connect() {
    fetch(this.urlValue)
      .then(/* ... */)
  }
}
```

### a. Definitions

Define values in a controller using the `static values` object. Put each value's name on the left and its type on the right.

```JS
export default class extends Controller {
  static values = {
    url: String,
    interval: Number,
    params: Object
  }

  // ...
}
```

### b. Types

A value's type is one of `Array`, `Boolean`, `Number`, `Object`, or `String`. The type determines how the value is transcoded between JavaScript and HTML.

| Type    | Encoded as...            | Decoded as...                         |
|---------|--------------------------|---------------------------------------|
| Array   | `JSON.stringify(value)`  | `JSON.parse(value)`                   |
| Boolean | `boolean.toString()`     | `!(value == "0" || value == "false")` |
| Number  | `number.toString()`      | `Number(value)`                       |
| Object  | `JSON.stringify(object)` | `JSON.parse(value)`                   |
| String  | Itself                   | Itself                                |

## <u>7. CSS Classes</u>

## <u>8. Using Typescript</u>
