# Transferring Data Between Angular Components: Best Practices

Transferring data between Angular components is a headache. 

Angular's strength is its modular structure. Each section of a web page&#151;headers, footers, forms, etc.&#151;is a component. Each component has its own HTML view, controller, CSS, and test files. The components fit together into a seamless view for the user.

(Many Angular developers refer to the controller as the component. I use "component" to mean the four files together. The controller is the logic file.)

This modular structure enables Angular to scale. AngularJS was easy to set up for a small web app, but when the app scaled up the controllers grew to thousands of lines. These controllers were spaghetti code, where tracing the path of a variable was like trying to follow a strand of pasta on your plate. 

In contrast, Angular's modular structure keeps controllers to typically a few hundred lines. Tracing a variable is usually straightforward as we can see a variable's value when it enters the component, how it changes in the component, and the value when it is sent to a another component.





The headache is that your web app likely has hundreds of components. Data is constantly transferred between components. Transferring data between components isn't simple. Most of your bugs will be in transferring data between components. If you have poor data transfer practices your web app will become unreadable, unmaintainable, and slow. This tutorial will teach you best practices for readable, maintainable, fast-running code.

### Parent and child components

Angular passes data between components via the HTML views. This might seem odd, as data is handled and transformed in the controllers.

The HTML view of a *parent component* includes a link to the HTML view of a *child component*. 

*parent.component.html*
```html
 <app-home-toolbar></app-home-toolbar>
```

Where the components are in the directory structure of the Angular project has nothing to do with parent-child relationships.

## The Basics

Angular has four ways to transfer data between components.

* `@Input` transfers data from a parent component to a child component.
