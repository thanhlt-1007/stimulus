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

#### - Registering Controllers Manually

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

## <u>2. Lifecycle Callbacks</u>

## <u>3. Actions</u>

## <u>4. Targets</u>

## <u>5. Outlets</u>

## <u>6. Values</u>

## <u>7. CSS Classes</u>

## <u>8. Using Typescript</u>
