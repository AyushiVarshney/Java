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
import {ElementRef, Directive, Input, HostListener, Renderer2} from '@anhular/core';

@Directive({
      selector: '[appHighlightOnHover]'
})
export class HighlightDirective {
      @Input highlightColor : string = 'yellow'; //default color
      constructor(private el: ElementRef, private renderer: Renderer2){}

      @HostListener('onmouseenter') onMouseEnter() {
            this.el.nativeElement.style.backgroundColor = this.highlightColor;
            //Alternative Approach Using Renderer2 (Safer)
            this.renderer.setStyle(this.el.nativeElement, 'background-color', this.hightlightColor);
      }

      @HostListener('onmouseleave') onMouseLeave() {
            this.el.nativeElement.style.backgroundColor = 'transparent';
      }
//Renderer2 is recommended because directly modifying the DOM using ElementRef can create security risks (e.g., XSS attacks).
//It provides an abstracted way to manipulate the DOM safely.

}

@Component({
      selector: 'app-my-app',
      templateUrl: './app-my-app.component.html', or template: `<p appHighlightOnHover> This is some text</>`,
      styleUrls: ['./app-my-app.component.scss']
})
//we can also pass diff color `<p appHighlightOnHover hightlightColor = 'skyblue'> This is some text</>`,
export class MyApp {}

```

# HostBinding and HostListener
HostBinding('value') myValue; is exactly the same as [value]="myValue"
And
HostListener('click') myClick(){ } is exactly the same as (click)="myClick()"

HostBinding and HostListener are written in directives and the other ones (...) and [..] are written inside templates (of components).
# what is subject, behaviour subject and reply subject
