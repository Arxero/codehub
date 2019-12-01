---
layout: post
title:  "Cancel Navigation with Confimation dialog"
date:   2019-11-28 18:50:32 +0200
categories: angular
author: Maverick
comments: true
---

In this tutorial you will be learning how to cancel navigation with confrimation pop-up dialog using can `CanDeactivate` interface and a guard for that matter. At the end I will show you as well how to make it with **generics** so you can use it on all of you components that need that.

[Finished Live Example](https://stackblitz.com/edit/angular-cancel-navigation-with-dialog)

[Finished Live Example - Generic Template](https://stackblitz.com/edit/angular-cancel-navigation-with-dialog-stgss1)

    ng new cancel-navigation-tutorial
    cd cancel-navigation-tutorial
    ng add @angular/material

Now let's generate two component which we are gonna use for navigation
    
    ng g c page1
    ng g c page2


Add to your project *angular material* if you want at least that is what I did to show better looking buttons if you wanna see how just see the stackblitz link above. 

After that just put those two link in your `app.component.html` file, so we can navigate to each one of them.

```html
<a mat-raised-button color="primary" routerLink="page1">Page 1</a>
<a mat-raised-button color="primary" routerLink="page2">Page 2</a>
```

Let's make as well the confirmation dialog component

    ng g c dialog

`dialog.component.html`
```html
<div mat-dialog-content>
  <p>{% raw %}{{ text }}{% endraw %}</p>
</div>
<div mat-dialog-actions>
  <button mat-button (click)="onYesClick()">Yes</button>
  <button mat-button (click)="onCancelClick()">Cancel</button>
</div>
```

`dialog.component.ts`
```typescript
export class DialogComponent {
  @Input() text: string;
  subject: Subject<boolean>;

  constructor(private dialogRef: MatDialogRef<DialogComponent>) { }

  onYesClick() {
    if (this.subject) {
      this.subject.next(true);
      this.subject.complete();
    }
    this.dialogRef.close(true);
  }

  onCancelClick() {
    if (this.subject) {
      this.subject.next(false);
      this.subject.complete();
    }
    this.dialogRef.close(false);
  }
}
```

Also make sure you put the **DialogComponent** in `entry components` array in your **NgModule**. Next we would need to make some condition based on which we want to cancel the navigation, usually this is some input change.

`page2.component.html`
```html
<mat-form-field>
  <input matInput placeholder="Your Name" (change)="isInput = true">
</mat-form-field>
```
Don't forget to define the **isInput** variable in the ts file.

Make a new file and call it `navigation.guard.ts` in it paste this:

```typescript
@Injectable({
  providedIn: 'root'
})
export class CustomConfirmGuard implements CanDeactivate<Page2Component> {
  confirmDlg: MatDialogRef<DialogComponent>;

  constructor(
    private dialog: MatDialog
  ) {}

  canDeactivate(component: Page2Component) {
    const subject = new Subject<boolean>();

    if (component.isInput) {
      this.confirmDlg = this.dialog.open(DialogComponent, { disableClose: true });
      this.confirmDlg.componentInstance.subject = subject;
      this.confirmDlg.componentInstance.text = 'Are you sure?';
      return subject.asObservable();
    }

    return true;
  }
}
```
Finally make sure you guard your **page2** route with our guard;
`app-routing.module.ts`

```typescript
const routes: Routes = [
  { path: 'page1', component: Page1Component },
  { path: 'page2', component: Page2Component, canDeactivate: [CustomConfirmGuard] },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```
And that's about it at this you have anything you need to use it as it is, but in the next part of this tutorial I will show you how to make a generic teplate out of this and possibly use it in bunch of other places without hesitation.

## Generic Template

Edit your `navigation.guard.ts` like that

```typescript
export interface ComponentGuard {
  isInput: boolean;
}

export class CustomConfirmGuard<T extends ComponentGuard> implements CanDeactivate<T> {
  confirmDlg: MatDialogRef<DialogComponent>;
  subject = new Subject<boolean>();
  text: string;

  constructor(protected dialog: MatDialog) {
  }

  canDeactivate(component: T) {
    if (component.isInput) {
      this.confirmDlg = this.dialog.open(DialogComponent, { disableClose: true });
      this.confirmDlg.componentInstance.subject = this.subject;
      this.confirmDlg.componentInstance.text = this.text;
      return this.subject.asObservable();
    }

    return true;
  }


}

@Injectable({
  providedIn: 'root'
})
export class Page2ConfirmGuard extends CustomConfirmGuard<Page2Component> {

  constructor(dialog: MatDialog) {
    super(dialog);
    this.text = 'Are you sure leaving Page 2?';
  }
}

@Injectable({
  providedIn: 'root'
})
export class Page1ConfirmGuard extends CustomConfirmGuard<Page1Component> {

  constructor(dialog: MatDialog) {
    super(dialog);
    this.text = 'Are you sure leaving Page 1?';
  }
}
```

Your `app-routing.module.ts`

```typescript
const routes: Routes = [
  { path: 'page1', component: Page1Component, canDeactivate: [Page1ConfirmGuard] },
  { path: 'page2', component: Page2Component, canDeactivate: [Page2ConfirmGuard] },
];
```

Your `page1.component.ts`
```typescript
export class Page1Component implements ComponentGuard {
  isInput: boolean = true;
}
```

Your `page2.component.ts`
```typescript
export class Page2Component implements ComponentGuard {
  isInput: boolean = false;
}
```
And finally you need to comment `this.subject.complete();` line in both functions of `dialog.component.ts`. With this template you will be able with minimal effort to use this in every component you like.
