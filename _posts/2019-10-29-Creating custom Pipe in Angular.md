---
layout: post
title:  "Creating a custom Pipe in Angular"
date:   2019-10-29 18:50:32 +0200
categories: angular
author: Maverick
comments: true
---

In this tutorial I will be showing you how to make a custom pipe in angular in the simpliest and shortest way possible.

[Finished Live Example](https://stackblitz.com/edit/creating-a-custom-pipe-in-angular)

    ng new pipe-tutorial
    cd pipe-tutorial

With that out of the way, lets generate our pipe using angular's cli. You can do it yourself of course by creating, a new file and importing it in `app.module.ts` manually.

    ng g pipe filter --skipTests=true

`--skipTests=true` will just won't create **.spec** file. Now as you saw by the name of the pipe I am inteding to make a filter pipe that is going to be filtering and array of elements in the UI by entered word. 

This is how our `filter.pipe.ts` file looks after adding some simple code:

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'filter'
})
export class FilterPipe implements PipeTransform {

  transform(items: any[], searchTerm: any, searchBy: string) {

    // when our serach is undefined or null
    if (!searchTerm) {
      return items;
    }

    // when there is partial or full match of the search term
    return items.filter(item => {
      const currentItem = item[searchBy];
      return currentItem.toString().toLowerCase().includes(searchTerm.trim().toLowerCase());
    });
  }
}

```

This is our `app.component.ts` file, just a simple array where we put some people.
```typescript
import { Component } from '@angular/core';
import { FilterPipe } from './filter.pipe';

export interface Person {
  name: string;
  age: number;
  country: string;
}

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})

export class AppComponent {
  people: Person[] = [];
  searchTerm: string;
  names = ['Maverick', 'Stanislav', 'Arxero', 'Feruchio', 'Mavericus', 'Arxiour'];

  constructor() {
    this.names.forEach((e, i) => this.people.push({
      name: e,
      age: i + 20,
      country: 'Bulgaria'
    }));
  }
}
```

Import `FormsModule` from

```typescript
import { FormsModule } from '@angular/forms';
```
and paste the html code in `app.component.html`

```html
<div>
  <input [(ngModel)]="searchTerm" placeholder="Filter">

  <ul>
    <li *ngFor="let person of people | filter: searchTerm: 'name'">
      {% raw  %} 
      {{person.name}} | {{person.age}} | {{person.country}}"
      {% endraw %}
    </li>
  </ul>
</div>
```

And that is all about it, I made also posible to search by a selected property of the person object by giving it in the html as second parameter to the pipe. `filter: searchTerm: 'name'"`



