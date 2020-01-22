---
layout: post
title:  "Reactive forms In Angular, example"
date:   2019-10-31 18:50:32 +0200
categories: angular
author: Maverick
comments: true
---

Reactive forms in angular are qute important and usefull that is a why you a as angular developer need to put them in your toolbox. So let's start with a basic impelentation of them.

[Finished Live Example](https://stackblitz.com/edit/reactive-forms-in-angular-examle)

    ng new reactive-forms-tutorial
    cd reactive-forms-tutorial

Now as first thing you would need to do is:
```typescript
import { ReactiveFormsModule } from '@angular/forms';
```
also in the **imports** array in the `app.module.ts` file
```typescript
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    ReactiveFormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
```

Let's lay down some simple code for the form in the `app.component.ts` to illustrate how you would create.
```typescript
import { Component, OnInit } from '@angular/core';
import { FormGroup, FormBuilder, Validators } from '@angular/forms';

export interface Person {
  name: string;
  age: number;
}

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent implements OnInit {
  addPersonForm: FormGroup;
  person: Person;

  constructor(private formBuilder: FormBuilder) {}

  ngOnInit(): void {
    this.addPersonForm = this.formBuilder.group({
      name: ['', [Validators.required, Validators.minLength(3), Validators.maxLength(20)]],
      age: ['', [Validators.required, Validators.min(18), Validators.max(101)]],
    });
  }

  // this is just a better way of getting form properties for showing error validation
  get name() {
    return this.addPersonForm.get('name');
  }
  get age() {
    return this.addPersonForm.get('age');
  }

  onSubmit(formData: Person) {
    this.person = formData;
  }

}
```
`app.component.html`
```html
<form [formGroup]="addPersonForm" (ngSubmit)="onSubmit(addPersonForm.value)">
  <div>
  
    <input placeholder="Name" formControlName="name">
    <div class="error" *ngIf="name.errors?.required && name.touched">
      Name is <strong>required</strong>
    </div>

    <div *ngIf="name.errors?.minlength && name.touched">
      Min Length of <strong>3</strong>
    </div>

    <div *ngIf="name.errors?.maxlength && name.touched">
      Max Length of <strong>20</strong> exceeded
    </div>
  </div>

  <div>
    <input placeholder="Age" formControlName="age">
    <div *ngIf="age.errors?.required && age.touched">
      Age is <strong>required</strong>
    </div>

    <div *ngIf="age.errors?.min && age.touched">
      Min Value of <strong>18</strong>
    </div>

    <div *ngIf="age.errors?.max && age.touched">
      Max Value of <strong>101</strong> exceeded
    </div>

  </div>

  <button [disabled]="addPersonForm.invalid" color="primary" type="submit">Submit</button>
</form>
```

And with this, we are done with implementing this simple example of angular reactive forms. Hopefully, you would be able to copy-paste and use it in your project.


