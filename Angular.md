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
import {ElementRef, Directive, Input, HostListener, Renderer2} from '@angular/core';

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
//we can also pass diff color `<p appHighlightOnHover [hightlightColor] = 'skyblue'> This is some text</>`,
export class MyApp {}

```

# HostBinding and HostListener
HostBinding('value') myValue used for changing state of host element; is exactly the same as [value]="myValue"
And
HostListener('click') myClick(){ } is used for listening to event; is exactly the same as (click)="myClick()"

HostBinding and HostListener are written in directives and the other ones (...) and [..] are written inside templates (of components).
```ts
import { Directive, HostBinding, HostListener, Input } from '@angular/core';

@Directive({
  selector: '[appHoverEffect]',
})
export class HoverEffectDirective {
  
  @Input() hoverColor: string = 'yellow';  // Default color

  @HostBinding('style.backgroundColor') backgroundColor: string;

  @HostListener('mouseenter') onMouseEnter() {
    this.backgroundColor = this.hoverColor;  // Change background color on mouse enter
  }

  @HostListener('mouseleave') onMouseLeave() {
    this.backgroundColor = null;  // Reset background color on mouse leave
  }
}
```

# what is SimpleChanges in Angular
Simplechanges is an interface that provides changes in Input properties os component. When angular detect changes in Input property it triggers ngOnChanges hook which receives argument of Type simplechanges which is a map object.
For example if my component has 2 input propeties , title and message then 
```ts
ngOnChanges(changes: SimpleChanges){
      if(changes['title']){//title has changed}
      if(changes['message']){//message has changed}
}
//Structure of SimpleCHanges
{
  title: SimpleChange {
    previousValue: 'Old Title',
    currentValue: 'New Title',
    firstChange: false
  },
  message: SimpleChange {
    previousValue: 'Old Message',
    currentValue: 'New Message',
    firstChange: false
  }
```
# What is RxJs
Its a library for reactive programming using Observables to perform asynchronous tasks. We can use for
1. Http Calls
2. State Management (ex : using Ngrx)
3. Handilng User events
4. handling real time updates
```ts
import {Observable} from 'rxjs';

const observable = new Observable(subscriber => {
      console.log('Hello from Observable');
      subscriber.next('Hello');
      subscriber.next('RxJs');
      subscriber.complete();
}); //By default observables are cold. Meaning is there are multiple subscribers, observables will run from start for every subscription

observable.subscribe(value => console.log(value));
observable.subscribe(value => console.log('I am here too ' + value));

Output: Hello from Observable
        Hello
        RxJs
        Hello from Observable
        I am here too Hello
        I am here too RxJs

```
We can share observables among multiple shared instance(singleton global instance) subscribers (Hot Observables)
```ts
const observable = new Observable(subscriber => {
      console.log('Hello from Observable');
      subscriber.next('Hello');
      subscriber.next('RxJs'); //here observable can emit multiple values like hello and rxjs while promise only resolves once and executes immedietly and store resolve function
      subscriber.complete();
}).pipe(share());
observable.subscribe(value => console.log(value));
observable.subscribe(value => console.log('I am here too ' + value));

Output: Hello from Observable
        Hello
        I am here too Hello
        RxJs
        I am here too RxJs
```
# what is subject, behaviour subject, reply subject and async subject:
Observables are unicasting and will emit data only when they are subscribed to. Each subscriber has own execution of observer. UseCase: For http calls
subject, behaviour subject, reply subject and async subject are hot observers. All subscribers share execution.

```ts
//Subject is both observer and observable
let mySubject = new Subject<String>();//In observer we have to pass subscriber here which emits values
mySubject.next('Hello'); //for subject we can do like this. this value will be missed by subscriber as subject has not been subscribed yet. UseCase: For real time updates

//Behaviour Subject: Requires an initial value and stores last emitted value. Late subscribers will not miss emitted value
//UseCase: For state management (theme setting, user authentication)
let bS = new BehaviousSubject<String>('Initial');
bS.subscribe(val => console.log(val)); //Initial
bs.next('First Update'); //Subscriber 1 receives First Update and behaviour subject store First Update
bs.next('Second Update');//Subscriber 1 receives Second Update and behaviour subject store Second Update

bs.subscribe(val => console.log('Second' + val)); //immedietly gets Second Update
bs.next('Third Upadate'); //both subscribers gets Third Update and behaviour subject stores Third Update

//Replay Subject: is like behaviour and also stores multiple previous values
let rS = new ReplySubject<number>(3);//define how many history to store
rS.next(1);
rS.next(2);
rS.next(3);
rS.next(4);//1 is deleted and [2,3,4] are stored
rS.subscribe(val => console.log(val)); // 2 then in next line 3 and in next line 4

//AsyncSubject: Stores last emmitted value. Subcribers will only receive value when complete is called.
let aS = new AsyncSubject<number>();
as.subscribe(val => console.log(val));

as.next(1); //subscriber will not receive anything
as.next(2); //subscriber will not receive anything and 2 will replace 1
as.next(3); //subscriber will not receive anything and 3 will replace 2

as.complete(); //3 will be printed

as.subscribe(val => console.log(val)); // second subscriber will also receive 3 and 3 will be printed
```

# What is explicit binding (call, apply, bind)
Process of explicitly setting value os this using call, apply and bind methods
```ts
const person = {
   name: 'Ayushi',
   age: 25,
   printPerson: function(gender) {
      console.log(`${this.name}'s age is ${this.age} and gender is ${gender}`);
   }
}

const person2 = {name: 'Parag',age:30};
person.printPerson.call(person2, 'male');

const bind = printPerson.bind(person, 'female');//bind regturns a function which we have to call
bind();

//apply is similiar to call, it just takes function arguments as array
```

# How you can do inheritance in java script
```ts
//1. Using Object.create()
const obj = {
   name:'Ayushi',
   print: function(){}
}
const obj2 = Object.create(obj);//obj2 will have all properties of obj

//2. Using prototype inheritence
//every thing in java is objects like array, functions etc and every object in java has prototype

function Document(title, author) {
   this.title = title;//creating a constructor function
   this.author = author;
}

Document.prototype.print = function(){
   console.log(`${this.title} by ${this.author}`); //document has a function print & each instance will share print rather than a new copy
}

function PDFDoc(title, author, size){
   Document.call(this, title, author);//calling parent constructor function
   this.size = size;
}

PDFDoc.prototype = Object.create(Document.prototype);//pdfdoc proto is having document proto
PDFDoc.prototype.contructor = PDFDoc; //giving back pdfdoc its constructor;

const doc = new PDFDoc('The Hobbit', 'Jimmy', 1024);
doc.print();//PDFDoc is able to access print function which is in Document proto

```
