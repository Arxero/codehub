---
layout: post
title:  "Simple store with BehaviorSubject - Angular"
date:   2020-01-23 13:50:32 +0200
categories: angular
author: Maverick
comments: true
---

Hello everyone, in the last post I showed you how to add Ngrx to your angular project, but I do realize that might not always be your use case for such a sophisticated state management solution, that's why in this we will take a look at BehaviorSubject and how to use it.

[Finished Live Example](https://stackblitz.com/edit/simple-store-with-behaviorsubject-angular)


    ng new angular-behavior-subject-tutorial
    cd angular-behavior-subject-tutorial

Now let's generate our state service

    ng g s state

`state.service.ts`
```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class StateService {
  private state$ = new BehaviorSubject<string>('');
  private value = 0;

  constructor() { }

  inc() {
    this.value++;
    this.state$.next(this.value.toString());
  }

  decrement() {
    this.value--;
    this.state$.next(this.value.toString());
  }

  finish() {
    this.state$.complete();
  }

  get state(): Observable<string> {
    return this.state$.asObservable();
  }
}
```

And in your `app.component.ts` you should subscribe just
```typescript
export class AppComponent {
  number: string;
  number$: Observable<string>;

  constructor(private stateService: StateService) {
    this.stateService.state.subscribe(x => {
      this.number = x;
    });

    this.number$ = this.stateService.state;
  }

  onIncrement() {
    this.stateService.inc();
  }

  onDecrement() {
    this.stateService.decrement();
  }

  onFinish() {
    this.stateService.finish();
  }
}
```

`app.component.html`
```html
<button (click)="onIncrement()">Add</button>
<button (click)="onDecrement()">Remove</button>
<button (click)="onFinish()">Finish</button>
<br>
<br>
<div>{% raw %}{{ number }}{% endraw %}</div>
<div>{% raw %}{{ number$ | async }}{% endraw %}</div>
```

And that is how to use simple state managment in Angular with BehaviorSubject, hope you found that useful.