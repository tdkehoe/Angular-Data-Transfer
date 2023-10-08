# Sharing Data Between Angular Components: Best Practices

## tl;dr

* Use `@Input()` and `@Output()` to share data between sibling components.
* Use the  `@Input()` getter/setter syntax to enable debugging and check if data is shared.
* Use Angular Signals in a service to share data between unrelated components.
* Send data to a service from a component controller function, with the data as parameters in the service function. Don't set up listeners in services to maintain current values of variables, i.e., don't try to maintain state in a service.
* To get data from a service, make a listener in `ngOnInit()`. Don't try to get data from a service in a function.
* Minimize database calls. Database calls cost time and money.
* Maintaining state, i.e., updating changes in a value through every component that uses the variable, is a non-trivial problem.
* When sharing objects between components you must tranform the object to get Angular's "dirty checking" to see that a value has changed in a property.
* When setting up data sharing between components, set up tracing data changes back to their sources for debugging.
* Make tests for data sharing paths.

## Why Angular data sharing between components is a headache

Let's rephrase each item in the previous section as a negative instead of a positive.

* `@Input()` and `@Output()` are overly complex, require writing too much code, and have too many things that can go wrong, especially with the getter/setter syntax.
* A function can't get data from a service. You can only listen for a service to broadcast data to listeners.
* The values in a database should always be the current state, i.e., the canonical values. But maintaining this state slows your app and can be expensive.
* Different components can have different values for the same variable, i.e., state isn't maintained.
* `@Input()` and `@Output()` sometimes just don't work. My guess is that Angular's "dirty checking" misses more than changes in objects.
* Tracing data changes back to their sources isn't built in to Angular and increase coding complexity.
* Writing tests for data sharing paths isn't simple.

Sharing data between Angular components is complex, unreliable, difficult to debug and test, and too often results in different components having different values for a shared variable (state isn't maintained). Most bugs are due to problems sharing data between components.

## My story...

My story starts in coding bootcamp in 2015 when I fell in love with AngularJS. AngularJS made setting up a small web app so easy. We had a big final exam in which we had seven hours to set up a CRUD app to specifications. (A CRUD app has a front end that reads and writes to a database in the back end. CRUD is an acronym for the Create, Read, Update, Delete database operations.)

We were required to use MongoDB as the database, running in Express on Node on the back end. Everyone else in my class used something called Handlebars to make the data appear in the front end. I used AngularJS. I finished in three and a half hours. My app looked better and had more features than my classmates' work. Only a quarter of the class passed this exam. 

I stopped loving AngularJS after I graduated and built a large project.  The controllers grew to thousands of lines. These controllers were spaghetti code, where tracing the path of a variable was like trying to follow a strand of pasta on your plate.

Angular (at first called Angular 2) was released a year later. Angular's strength is its modular structure. In AngularJS, each web page or view had a controller. In Angular, each section of a view--headers, footers, forms, etc.--is a component. Each component has its own HTML view template, controller, CSS, and test files. The components fit together into a seamless view for the user.

Angular's modular structure keeps controllers to a few hundred lines. Tracing a variable is straightforward as we can see a variable's value when it enters the component, how it changes in the component, and the value when it is sent to a another component.

The headache is that a large web app has dozens or hundreds of components. Data is shared between components. Sharing data between components isn't simple. Most of your bugs will be in sharing data between components. If you have poor data sharing practices your web app will become unreadable, unmaintainable, and slow. This tutorial will teach you best practices for readable, maintainable, fast code.

### Nomenclature

Angular is built with components. A component has four files. What are these files called? Following the [Angular documentation](https://angular.io/guide/component-overview):

* An HTML template
* A TypeScript class
* A CSS stylesheet
* A TypeScript test file

Following the [model-view-controller](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) (MVC) nomenclature, the HTML template is the "view" and the TypeScript class is the "controller".

It's incorrect to call the TypeScript class the "component". The "component" is the four files together.

At the expense of verbosity I'll choose clarity and call the first file the "HTML view template" and the second file the "TypeScript class controller".

## My Story...First, Sharing Data Via Services

I read the [Angular documentation](https://angular.io/guide/inputs-outputs) about the four ways to share data between components. Using a service sounded like the best choice. Services can share data from parent to child components, from child to parent components, or between unrelated components. Plus, in AngularJS the only way to share data between controllers was via services so I was familiar with this method.

I built a service for sharing data between components. As I built my project I added getter/setters for variables as needed. This grew to hundreds of getter/setter code blocks.

*share-data.service.ts*
```js
  setShowForm(showForm: boolean = false): void {
    this.showForm.next(showForm);
  }
  getShowForm(): Observable<boolean> {
    return this.showForm.asObservable();
  }
```

Setting the data wasn't difficult. I just called the service from a TypeScript class controller:

*my-component.component.ts*
```js
this.shareDataService.setShowForm(this.showForm);
```

Getting the data required an Observable.:

*another-component.ts*
```js
 this.shareDataService.getShowForm().subscribe((showForm: boolean) => {
      this.showForm = showForm;
    });
```

Things can go wrong with this data sharing structure.

The Observable can only be in `ngOnInit()`. It gets the value at page load and then the Observable changes the local value whenever the data in the service changes. This should work with synchronous data but it's a disaster with async data such as database calls. You never know whether the local data is the current value in the database or a previous value.

Even with synchronous data, tracing bugs was a nightmare. If more than one component can set the data with `this.shareDataService.setShowForm(this.showForm);`, and you have anomolous data showing up in the observing component, then you have to figure out which component is setting the buggy data. Later I'll give a tip that makes this easier.

In the component that consumes the data, several functions might change the local variable.

To chase down a bug I had to:

*Log the data in every component that set the data.
*Log the data in the service.
*Log the data in Observer that gets the data.
*Log the data everywhere this data is consumed or altered in the component that gets the data.

This can easily be a dozen places in your code. With hundreds of variables shared between dozens of components, chasing down bugs was a nightmare. In a word, `state`. Keeping all my components in the same state was a nightmare.

## My Story...Second, Sharing Data Via the Database

If we want to maintain a canonical source for our data, let's not share data between services. Instead, whenever data changes, write it to the database. Whenever you need data, get it from the database.

My app slowed to a crawl. Each database call might be fifty or one hundred milliseconds (on a good day) but with dozens of database calls every click made the user wait ten seconds or more.

Lesson learned: minimize your database calls.

## My Story...Third, Sharing Data Via @Input and @Output

I refactored my project again, using `@Input` to share data from parent to child components and `@Output` to share data from child to parent components. This is the `best practices` way to share data between components in Angular but there's plenty of complexity and more than a few pitfalls. This tutorial aims to make this help you aaoid headaches.

## The Basics

Angular has four ways to transfer data between components.

* The `@Input()` decorator transfers data from a parent component to a child component.
* The `@Output()` decorator transfers data from a child component to a parent component.
* The `@ViewChild()` decorator enables a parent component to see a value in a child component.
* Data can be transferred between unrelated components via services. This includes the new Signals method.

### Parent and child components

Angular passes data between components via the HTML files. This might seem odd, as data is handled and transformed in the controllers. Angular makes us forget that the web is ultimately HTML code.

The HTML file of a *parent component* includes a link to the *child component*. 

*parent.component.html*
```html
 <app-home-toolbar></app-home-toolbar>
```

Where the components are in the directory structure of the Angular project has nothing to do with parent-child relationships. This relationship is only about links in HTML files.

## The `@Input()` and `@Output()` decorators

The `@Input()` and `@Output()` decorators are the workhorses of Angular data transfer. Both require code in three files:

* Declare variables in the parent TypeScript class controller.
* Write transfer code in the parent HTML view template.
* Use the `@Input()` and `@Output()` decorators when declaring the variables in the child TypeScript class controllers.

And each has more code. The downsides of these methods is that they are complex, especially when you're sending data between sibling components via a parent component. This requires lots of code in multiple places in four (or more) files. You get many opportunites to make a typo.

Worse, these methods sometimes fail to share values. Below I will explain Angular's "dirty checking" causes these methods to fail and how to fix this. But sometimes a data change in one component fails to show up in a sibling component and I can't find any typo or mistake. I suspect that there are other problems with "dirty checking" that I'm unaware of. Or there's some other hidden "gotcha" that I haven't yet learned.

### `@Input()` in the parent HTML view template

In the parent HTML view template you write the transfer code in the child component link.

*parent.component.html*
```html
  <app-child-component></app-child-component>
```

The `@Input()` code (to transfer data from the parent component to the child component) is written with square brackets[]. The code is simple, just the child variable name in the square brackets, an equals, and the parent variable name in quotations. The child variable name and the parent variable are usually the same.

*parent.component.html*
```html
<app-child-component>
[myVariable]="myVariable"
</app-child-component>
```

*Tip:* Variable names should start with a lowercase letter. If your data isn't transferring, check that you have lowercase letters starting both sides of this code.

*Tip:* If your data isn't transferring between components, check the parent HTML file first. I alphabetize my `@Input` code to make them easier to find.

### `@Input()` in the child TypeScript class controller

Import `Input` into the child TypeScript class controller.

*child.component.ts*
```js
import { Component, EventEmitter, Input, OnInit, Output } from '@angular/core';
```

Then precede the variable declaration with the `@Input()` decorator. (Decorators are functions invoked with the @ symbol that precede a class, method, or property. Variables are properties of a component.)

*child.component.ts*
```js
export class ItemDetailComponent {
  @Input() myVariable: boolean = false;
}
```

### `@Input()` in the child HTML view template

Display the value in the HTML view template:

*child.component.html*
```html
<p>
  The current value is: {{myVariable}}
</p>
```

### Trigger actions in the child TypeScript class controller when `@Input()` changes

Use `OnChanges()` in the child component to trigger actions when the a value in the parent component changes.

*child.component.ts*
```js
import { AfterViewInit, Component, OnChanges, OnInit, SimpleChanges, ViewChild } from '@angular/core';

export class ChildComponent implements OnChanges {
  ngOnChanges() {
    if (changes['myVariable']) {
      this.myFunction(myVariable);
    }
  }
}
```

### `@Input()` getter/setter

`@Input()` has an alternate syntax:

*child.component.ts*
```js

```



### Testing `@Input()`

### What could possibly go wrong with `@Input()`?

In a word, objects. 

`@Input()` will detect changes in [TypeScript primitives](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html): *string*, *number*, *boolean*, *bigint*, and *symbol*.

That leaves *array*, *any*, *functions*, *objects*, *unions*, *interfaces*, *null*, and *undefined*.

`@Input()` will not detect changes in values in objects. This is because Angular's "dirty checking" looks at the structure of objects, not the values. The structure doesn't change so `@Input()` doesn't pass the value. Fix this by making a new object whenever a value changes.

*child.component.ts*
```js
this.myObject.myProperty = true;
this.myObject = [...new Set(myObject)];
```

## The `@Output()` decorator

The `@Output()` decorator is used to transfer data from the child component to the parent component.

### `@Output()` in the parent HTML view template
The `@Output()` decorator is written with parentheses() in the parent HTML view template. The code is similar to `@Input()` code but the parent (right) side is a function, with `$event` as the parameter. 

*Caution:* Capitalization counts! Variable names should start with a lowercase letter. Function names should start with an Uppercase letter. `@Input()` HTML connections start with lowercase letters on both sides. `@Output()` HTML connections start with a lowercase letter on one side and an uppercase letter on the other side. If your data isn't transferring between components, check that you capitalized these codes correctly.

*parent.component.html*
```html
<app-child-component>
   (myVariable)="MyVariableEventHandler($event)"
</app-child-component>
```

The below code will transfer data both ways between the parent and child components.

*parent.component.html*
```html
<app-child-component>
   (myVariable)="MyVariableEventHandler($event)"
   [myVariable]="myVariable"
</app-child-component>
```

*Tip:* If your data isn't transferring between components, check the parent HTML view template first. I put the `@Output()` codes above the `@Input()` codes. I alphabetize each set of codes. As you scale your app you can get thirty, forty, or more of these codes. You must be able to see these codes without squinting.

### `@Output()` in the child TypeScript class controller

Import `EventEmitter` and `Output` into the child TypeScript class controller.

*child.component.ts*
```js
import { Component, EventEmitter, Input, OnInit, Output } from '@angular/core';
```

Then precede the variable declaration with the `@Output()` decorator.

*child.component.ts*
```js
export class DaughterComponent {
  @Output() myVariableEvent: EventEmitter<boolean> = new EventEmitter<boolean>();
}
```

You don't need to declare the type on both sides of the assignment operator. The right side is sufficient.

This syntax isn't a simple variable declaration as we had with `@Input()`. Instead of `myVariable1` we declare `myVariableEvent`. Then the value we assign this variable is a [EventEmitter](https://angular.io/api/core/EventEmitter).

`EventEmitter` is used only in `@Output()` directives. It extends [RxJS Subject](https://rxjs.dev/api/index/class/Subject) for Angular.

Now you can emit events when data changes.

*child.component.ts*
```js
async goToNextWord(): Promise<void> {
    this.myVariableEvent.emit(false);
}
```

This will emit `false` to the parent HTML view template, which shares this data with the parent TypeScript class controller.

### `@Output()` in the child HTML view template

We can make a button with a `(click)` event binding in the child HTML view template that will fire the `@Output()` event in the child TypeScript class controller.

*child.component.html*
```html
<label for="item-input">Add an item:</label>
<input type="text" id="item-input" #newItem>
<button type="button" (click)="addNewItem(newItem.value)">Add to parent's list</button>
```

### `@Output()` in the parent TypeScript class controller

In the parent TypeScript class controller, 

*parent.component.ts*
```js
  myVariableEventHandler(myVar: boolean) {
    this.myVariable = myVar;
  };
```

This will listen for emitted events from the child component and put the data on a local variable.

### Testing the `@Output()` decorator


