---
layout: post
title:  "Angular project with just Ngrx and Material"
date:   2020-01-22 13:50:32 +0200
categories: angular
author: Maverick
comments: true
---

In today's short tutorial I will show you how to install Ngrx and Angular material in a brand new project, this would be very helpful for new people to understand Ngrx and material is just something additional.

[GitHub repo](https://github.com/Arxero/Angular-project-with-just-Ngrx-and-Material)

[My follow along repo of Angular Ngrx Course](https://github.com/Arxero/angular-ngrx-course)


    ng new angular-with-ngrx-tutorial
    cd angular-with-ngrx-tutorial
    ng add @angular/material

Now install Ngrx packages as well

    ng add @ngrx/store
    npm install @ngrx/effects
    npm install @ngrx/router-store
    npm install @ngrx/store-devtools
    npm install @ngrx/entity
    ng add @ngrx/schematics
    npm install --save-dev ngrx-store-freeze

`app.module.ts`
```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { StoreModule } from '@ngrx/store';
import { reducers, metaReducers } from './reducers';

import { EffectsModule } from '@ngrx/effects';
import { environment } from 'src/environments/environment';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { StoreRouterConnectingModule } from '@ngrx/router-store';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    BrowserAnimationsModule,
    StoreModule.forRoot(reducers, {
      metaReducers,
      runtimeChecks: {
        strictStateImmutability: true,
        strictActionImmutability: true
      }
    }),
    EffectsModule.forRoot([]),
    StoreModule.forRoot(reducers, { metaReducers }),
    !environment.production ? StoreDevtoolsModule.instrument() : [],
    StoreRouterConnectingModule.forRoot({ stateKey: 'router' }),
    EffectsModule.forFeature([PeopleEffects])
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

By that time you should have `reducers` folder made automatically with `index.ts` file in it in there you can put my example of **people state**.

`index.ts`
```typescript
export interface Person {
  name: string;
  age: number;
}

// this is where you put all you different states of ngrx
export interface State {
  people: PeopleState;
}

// and this is one particular state from many
export interface PeopleState {
  people: Person[];
}

export const reducers: ActionReducerMap<State> = {
  people: peopleReducer
};

export const initialPeopleState: PeopleState = {
  people: []
} as PeopleState;

export function peopleReducer(
  state = initialPeopleState,
  action: PeopleActions): AppState {
  switch (action.type) {
    case PeopleActionTypes.LoadPeoplesSuccess:
      return {
        ...state,
        people: action.payload.data
      } as any;
    default:
      return state;
  }
}
```
Now make foders `actions`, `effects` and `selectors`

    ng g action actions/people

Leave it as it is generated it's okay like that

    ng g effect effects/people --module app.module

`effects/people.effects.ts`
```typescript
import { Injectable } from '@angular/core';
import { Actions, Effect, ofType } from '@ngrx/effects';
import { Store } from '@ngrx/store';
import { PeopleState, Person } from '../reducers';
import { LoadPeoples, PeopleActionTypes, LoadPeoplesSuccess } from '../actions/people.actions';
import { defer, of, Observable } from 'rxjs';
import { tap } from 'rxjs/operators';



@Injectable()
export class PeopleEffects {



  constructor(
    private actions$: Actions,
    private store: Store<PeopleState>) { }

  @Effect({ dispatch: false })
  logout$ = this.actions$.pipe(
    ofType<LoadPeoples>(PeopleActionTypes.LoadPeoples),
    tap(() => {
      try {
        const people: Person[] = [
          {
            name: 'Maverick',
            age: 26,
          } as Person
        ];
        this.store.dispatch(new LoadPeoplesSuccess({ data: people }));
      } catch (error) {

      }
    })
  );
}
```

And everything seem to be wired up now let's show some people in `app.component.html`
```html
<div *ngFor="let item of people$ | async">
   {% raw %}{{ item.name }} {{ item.age }}{% endraw %}
</div>
```

Also don't forget to add this to your `app.component.ts`
```typescript
import { Observable } from 'rxjs';
import { LoadPeoples } from './actions/people.actions';
import { PeopleState, Person } from './reducers/index';
import { Component, OnInit } from '@angular/core';
import { Store, select } from '@ngrx/store';
import { selectPeople } from './selectors/people.selectors';
import { map } from 'rxjs/operators';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  people$: Observable<Person[]>;

  constructor(private store: Store<PeopleState>) {}

  ngOnInit(): void {
    this.store.dispatch(new LoadPeoples());

    this.people$ = this.store.pipe(
      select(selectPeople)
    );
  }
}
```

And finally `ng serve` should show you the people on home page.