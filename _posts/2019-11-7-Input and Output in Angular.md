---
layout: post
title:  "Input and Output in Angular"
date:   2019-10-31 18:50:32 +0200
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
Also don't forget to let know the `child component` to expect what we give him by adding this.

```typescript
export class ChildComponent {
@Input() names: string[];
}
```
and the html
```html
<div *ngFor="let name of names">
  <br>
   {% raw  %} 
    {{ name }}
   {% endraw %}
  <button>Delete</button>
</div>
```
And with this the input part is done, simple as that. Now let's do the Output.

## Output
