# Decorators in Angular
Decorators in angular are functions that takes metadata and attach it to class, method, properties.
```ts
@Component      Marks an angular component
@Injectable     Marks a service for dependency Injection
@Input          From Parent to child
@Output         From Child to Parent
@HostListener   Allows you to listen to DOM events
@Directive      Define an angular Directive
@NgModule       Defines an angular module
```
# What is Directive and why do you need custom directive
is a special class that lets you extend behaviour of elements in DOM.
1. Component Directive : @Component (Every component in Angular is Directive with view.)
2. Structure Directive: *ngFor, *ngIf, *ngSwitch
3. Attribute Directive: ngClass, ngStyle, appCustomDirective

We need custom directive to encapsulate reusable behaviours, enhance user interaction, change existing elements dynamically.
```ts
@Directive({
      selector: '[app]
})
# what is subject, behaviour subject and reply subject
