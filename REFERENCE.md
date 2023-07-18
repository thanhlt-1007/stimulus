## <u>1. Controllers</u>

A controller is the basic organizational unit of a Stimulus application.

```JS
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {

}
```

Controller are instances of JavaScript classes that you define in your application. Each controller class inherits from the `Controller` base class exported by the `@hotwired/stimulus` module.

## a. Properties

Every controller belongs to a Stimulus `Application` instance and is associated with an HTML element. Within a controller class, you can access the controller's:

- application, via the `this.application` property

- HTML element, via the `this.element` property

- identifier, via the `this.identifier` property

## <u>2. Lifecycle Callbacks</u>

## <u>3. Actions</u>

## <u>4. Targets</u>

## <u>5. Outlets</u>

## <u>6. Values</u>

## <u>7. CSS Classes</u>

## <u>8. Using Typescript</u>
