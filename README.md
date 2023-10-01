# Sharing Data Between Angular Components: Best Practices

My story starts in coding bootcamp in 2015 when I fell in love with AngularJS. AngularJS made setting up a small web app so easy. We had a big final exam in which we had seven hours to set up a CRUD app to specifications. (A CRUD app has a front end that reads and writes to a database in the back end. CRUD is an acronym for the Create, Read, Update, Delete database operations.)

We were required to use MongoDB as the database, running in Express on Node on the back end. Everyone else in my class used something called Handlebars to make the data appear in the front end. I used AngularJS. I finished in three and a half hours. My app looked better and had more features than my classmates' work. Only a quarter of the class passed this exam. 

I stopped loving AngularJS after I graduated and built a large project.  The controllers grew to thousands of lines. These controllers were spaghetti code, where tracing the path of a variable was like trying to follow a strand of pasta on your plate.

Angular (at first called Angular 2) was released a year later. Angular's strength is its modular structure. In AngularJS, each web page or view had a controller. In Angular, each section of a view--headers, footers, forms, etc.--is a component. Each component has its own HTML view, controller, CSS, and test files. The components fit together into a seamless view for the user.

(Many Angular developers refer to the controller as the component. I use "component" to mean the four files together. The controller is the logic file. I will try to use the phrase "component controller" as you might be used to hearing the controller referred to as the component.)

Angular's modular structure keeps controllers to a few hundred lines. Tracing a variable is straightforward as we can see a variable's value when it enters the component, how it changes in the component, and the value when it is sent to a another component.

The headache is that a large web app has dozens or hundreds of components. Data is shared between components. Sharing data between components isn't simple. Most of your bugs will be in sharing data between components. If you have poor data sharing practices your web app will become unreadable, unmaintainable, and slow. This tutorial will teach you best practices for readable, maintainable, fast code.

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

Setting the data wasn't difficult. I just called the service from a component controller:

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

Even with synchronous data, tracing bugs was a nightmare. If more than one component can set the data with `this.shareDataService.setShowForm(this.showForm);`, and you have anomolous data showing up in the observing component, then you have to figure out which component is setting the buggy data. 

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

Where the components are in the directory structure of the Angular project has nothing to do with parent-child relationships. This relationship is all about links in HTML files.

### The `@Input` decorator

The `@Input` and `@Output` decorators are the workhorses of Angular data transfer. Both require code in three files:

* Declare variables in the parent component controller.
* Write transfer code in the parent HTML file.
* Use the `@Input` and `@Output` decorators when declaring the variables in the child component controller.

You don't have to import anything or do anything special when you declare your variables in the parent component controller.

#### `@Input` in the parent HTML file

In the parent HTML file you write the transfer code in the child component link.

*parent.component.html
```html
  <app-child-component></app-child-component>
```

The `@Input` code (to transfer data from the parent component to the child component) is written with square brackets[]. The code is simple, just the child variable name in the square brackets, an equals, and the parent variable name in quotations. The child variable name and the parent variable are usually the same.

*parent.component.html
```html
<app-child-component>
[myVariable]="myVariable"
</app-child-component>
```

*Tip:* Variable names should start with a lowercase letter. If your data isn't transferring, check that you have lowercase letters starting both sides of this code.

*Tip:* If your data isn't transferring between components, check the parent HTML file first. I alphabetize my `@Input` code to make them easier to find.

#### `@Input` in the child component controller

You must import `Input` into the parent component controller.

*parent.component.ts*
```js
import { Component, EventEmitter, Input, OnInit, Output } from '@angular/core';
```

Then precede the variable declaration with the `@Input()` decorator. (Decorators are functions invoked with the @ symbol that precede a class, method, or property. Variables are properties of a component.)

```js
export class ItemDetailComponent {
  @Input() item = ''; // decorate the property with @Input()
}
```

### The `@Output()` decorator

The `@Output` code (to transfer data from the child component to the parent component) is written with parentheses(). The code is similar to `@Input` code but the parent (right) side is a function, with `$event` as the parameter. 

*Caution:* Capitalization counts! Variable names should start with a lowercase letter. Function names should start with an Uppercase letter. `@Input` codes start with lowercase letters on both sides. `@Output` codes start with a lowercase letter on one side and an uppercase letter on the other side. If your data isn't transferring between components, check that you capitalized these codes correctly.

*parent.component.html
```html
<app-child-component>
(myVariable)="MyVariableEventHandler($event)"
</app-child-component>
```

The below code will transfer data both ways between the parent and child components.

*parent.component.html
```html
<app-child-component>
(myVariable)="MyVariableEventHandler($event)"
[myVariable]="myVariable"
</app-child-component>
```

*Tip:* If your data isn't transferring between components, check the parent HTML file first. I put the `@Output` codes above the `@Input` codes. I alphabetize each set of codes. As you scale your app you can get thirty, forty, or more of these codes. You must be able to see these codes without squinting.



