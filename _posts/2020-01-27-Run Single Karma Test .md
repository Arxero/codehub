---
layout: post
title:  "Run Single Karma Test - Angular"
date:   2020-01-27 13:50:32 +0200
categories: angular
author: Maverick
comments: true
---

In this post, you will be learning how to start a single unit test with Karma test runner in Angular.

    ng new angular-single-karma-test-tutorial
    cd angular-single-karma-test-tutorial

I know you can do just run an `ng test` command and it will run all of the spec files it finds and the tests in it, but for now, I would like to run just one that is made by me.

[Helpful link](https://stackoverflow.com/questions/41132933/running-a-single-test-file)

`custom-test.spec.ts`
```typescript
import { TestBed, async } from '@angular/core/testing';
import { RouterTestingModule } from '@angular/router/testing';
import { AppComponent } from './app.component';

describe('AppComponentCustom', () => {
  beforeEach(async(() => {
    TestBed.configureTestingModule({
      imports: [
        RouterTestingModule
      ],
      declarations: [
        AppComponent
      ],
    }).compileComponents();
  }));

  it('should sum properly', () => {
    const fixture = TestBed.createComponent(AppComponent);
    const app = fixture.debugElement.componentInstance;
    expect(app.sum(1, 3)).toEqual(4);
  });

});
```

`app.component.ts`
```typescript
export class AppComponent {
  sum(n1: number, n2: number) {
    return n1 + n2;
  }
}
```

Now put `f` in front of `describe` to focus only on that describe like `describe` and `f` in front of `it` to focus only on that test single test `fit`.

    ng test

And it should all work out for you, testing a single file, this will still show you all test but run only one. If you want to run only a single file just edit your `test.ts` with the path of your single file in my case is: `./src/app/custom-test.spec.ts`.

Just replace
```typescript
const context = require.context('./', true, /\.spec\.ts$/);
```
in `test.ts` with 
```typescript
const context = require.context('./', true, /custom-test\.spec\.ts$/);
```
and run again `ng test`.
And with that, you should have enough knowledge to test our components or classes by yourself in Angular.