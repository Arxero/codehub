---
layout: post
title:  "Input and Output in Angular"
date:   2019-11-7 18:50:32 +0200
categories: angular
author: Maverick
comments: true
---

In this post we will take a look at how to send data towards a component down the line and then get the changed data back to the parent one using [EventEmitter](https://angular.io/api/core/EventEmitter).

[Finished Live Example](https://stackblitz.com/edit/input-and-output-in-angular)

    ng new input-output-tutorial
    cd input-output-tutorial

Now let's make one child component that we are going to use for both decorators.

    ng g c child --skipTests=true

First of all we are going to send from `app.component.ts` to `child.component.ts` some array of strings that would be rendered in the view with a button for interaction.

```typescript
export class AppComponent {
  names: string[] = ['Maverick', 'Stanislav', 'Arxero', 'Feruchio', 'Mavericus', 'Arxiour'];
}
```
This is how we send our data `[names]` is what the child component expects and `names` is referring to our array in the parent. 
```html
<app-child [names]="names"></app-child>
```
Also don't forget to let know the `child.component.ts` to expect what we give him by adding this.

```typescript
export class ChildComponent {
  @Input() names: string[];
}
```
and the html, this will just render the names.
```html
<div *ngFor="let name of names">{% raw %}{{ name }}{% endraw %}</div>
```
And with this the input part is done, simple as that. Now let's do the Output.

## Output

Now let's add a button which on click will remove the selected element

`child component`

```html
<div *ngFor="let name of names">
  {% raw %}{{ name }}{% endraw %}
  <button (click)="onRemove(name)">Delete</button>
</div>
```

and handle the `onRemove` function in the **ts** file.
```typescript
export class ChildComponent {
  @Input() names: string[];

  onRemove(name: string) {
    this.names = this.names.filter(x => x !== name);
  }
}
```

I am doing this in order to have something meaningful to sent to the parent component, which will be the name of the deleted element.
Introduce the `Output` decorator like so:

```typescript
@Output() deletedName: EventEmitter<string> = new EventEmitter();
```

and then just add emit the name value to the parent component

```typescript
  onRemove(name: string) {
    this.names = this.names.filter(x => x !== name);
    this.deletedName.emit(name);
  }
```
`app component (parent)`

```html
<app-child [names]="names" (deletedName)="onDeletedName($event)"></app-child>
<br>

<div *ngIf="deletedName">
  Deleted Name is: {% raw %}{{ deletedName }}{% endraw %}
</div>
```

We just added a function which will receive what we sent

```typescript
export class AppComponent {
  names: string[] = ['Maverick', 'Stanislav', 'Arxero', 'Feruchio', 'Mavericus', 'Arxiour'];
  deletedName: string;

  onDeletedName(name: string) {
    this.deletedName = name;
  }
}
```

And that's pretty much it to input and output in angular, as simple as it gets. Feel free to add a comment if you think you can help someone with something.