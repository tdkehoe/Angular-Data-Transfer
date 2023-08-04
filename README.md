# Transferring Data Between Angular Components: Best Practices

The biggest headache for Angular developers is transferring data between components. 

Angular's strength is its modular structure. Views are built from components. Each component is typically a few hundred lines, with its own HTML view, controller, CSS, and test files. (Many Angular developers refer to the controller as the component. I use "component" to mean the four files together. The controller is the logic file.)

This modular structure enables Angular to scale. Us "old guys" who used AngularJS remember how easy it was to set up a small web app, but when the app scaled up the controllers got to be thousands of lines. These controllers were spaghetti code, where tracing the path of a variable was like trying to follow the path of a single strand of pasta on your plate. In contrast, Angular's modular structure makes me think of Angular as the freight train of front-end frameworks. Other frameworks may feel like sports cars but Angular is what you want for large, data-heavy web apps.

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
