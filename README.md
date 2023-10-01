# Sharing Data Between Angular Components: Best Practices

Sharing data between Angular components is a headache. 

Angular's strength is its modular structure. Each section of a web page--headers, footers, forms, etc.--is a component. Each component has its own HTML view, controller, CSS, and test files. The components fit together into a seamless view for the user.

(Many Angular developers refer to the controller as the component. I use "component" to mean the four files together. The controller is the logic file. I will try to use the phrase "component controller" as you might be used to hearing the controller referred to as the component.)

This modular structure enables Angular to scale. I fell in love with AngularJS because it made setting up a small web app so easy. I stopped loving AngularJS when my project scaled up and the controllers grew to thousands of lines. These controllers were spaghetti code, where tracing the path of a variable was like trying to follow a strand of pasta on your plate. 

In contrast, Angular's modular structure keeps controllers to a few hundred lines. Tracing a variable is straightforward as we can see a variable's value when it enters the component, how it changes in the component, and the value when it is sent to a another component.

The headache is that a large web app has dozens or hundreds of components. Data is constantly shared between components. Sharing data between components isn't simple. Most of your bugs will be in sharing data between components. If you have poor data sharing practices your web app will become unreadable, unmaintainable, and slow. This tutorial will teach you best practices for readable, maintainable, fast code.

## My Story...

I read the [Angular documentation](https://angular.io/guide/inputs-outputs) about the four ways to share data between components. Using a service sounded like the best choice. Service can share data from parent to child components, from child to parent components, or between unrelated components. Plus, in AngularJS the only way to share data between controllers was via services, so I was familiar with this method.

I built a service for sharing data between components. As I built my project I added getter/setters for variables as needed. This grew to hundreds of getter/setter code blocks.

*share-data.service.ts*
```js
private sentenceTranslation = new Subject<string | null>();
setSentenceTranslation(sentenceTranslation: string | null) {
    this.sentenceTranslation.next(sentenceTranslation);
}
  getSentenceTranslation() {
    return this.sentenceTranslation.asObservable();
}
```

Setting the data wasn't difficult. I just called the service from a component controller:

*my-component.component.ts*
```

```




### Parent and child components

Angular passes data between components via the HTML files. This might seem odd, as data is handled and transformed in the controllers. Angular makes us forget that the web is ultimately HTML code.

The HTML file of a *parent component* includes a link to the *child component*. 

*parent.component.html*
```html
 <app-home-toolbar></app-home-toolbar>
```

Where the components are in the directory structure of the Angular project has nothing to do with parent-child relationships. This relationship is all about links in HTML files.

## The Basics

Angular has four ways to transfer data between components.

* The `@Input()` decorator transfers data from a parent component to a child component.
* The `@Output()` decorator transfers data from a child component to a parent component.
* The `@ViewChild()` decorator enables a parent component to see a value in a child component.
* Data can be transferred between unrelated components via services.

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


export class ItemDetailComponent {
  @Input() item = ''; // decorate the property with @Input()
}


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



