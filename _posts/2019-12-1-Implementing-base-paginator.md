---
layout: post
title:  "Implementing base paginator"
date:   2019-12-01 13:50:32 +0200
categories: angular
author: Maverick
comments: true
---

In the following text you will be learning how to make your angular material paginator reusable with data source service and interface and generic classs that implements it.

[Finished Live Example](https://stackblitz.com/edit/angular-base-paginator)

    ng new base-paginator-tutorial
    cd base-paginator-tutorial
    ng add @angular/material


Let's start by making our interface and the generic class that will implement the paginator which we will make next.

`DataSource.ts`
```typescript
import { Observable, BehaviorSubject } from 'rxjs';

export interface DataSourceBase {
  length: Observable<number>;
  setPageSizeAndIndex(pageIndex: number, pageSize: number): void;
}

export interface DataSource<T> extends DataSourceBase {
  items: Observable<T[]>;
}

export class DataSourceIMPL<T> implements DataSource<T> {
  itemsBS: BehaviorSubject<T[]> = new BehaviorSubject<T[]>(null);
  lengthBS: BehaviorSubject<number> = new BehaviorSubject<number>(null);

  setPageSizeAndIndex(pageIndex: number, pageSize: number): void {
    throw new Error('Method not implemented.');
  }

  get items() {
    return this.itemsBS.asObservable();
  }

  get length() {
    return this.lengthBS.asObservable();
  }

}
```

Something that is worth mentioning is we will use `setPageSizeAndIndex()` method for getting the new data from a backend service or from some array for example, by giving it the values from the user comming from the **pagevent** and `onPageChange()` method in the paginator component. Also we are using **Observable** so we can subscribe for the length and new items in the angular component (app.component in this case) that we are going to use the paginator.

And now let's make our paginator component that we are going to reuse over and over again.

    ng g c base-paginator

`base-paginator.component.html`
```html
<mat-paginator
[length]="length"
[pageSize]="pageSize"
[pageSizeOptions]="pageSizeOptions"
(page)="onPageChange($event)"
>
</mat-paginator>
```
`base-paginator.component.ts`
```typescript
export class BasePaginatorComponent {
  _dataSource: DataSourceBase;

  @Input() set dataSource(value: DataSourceBase) {
    this._dataSource = value;
    this._dataSource.setPageSizeAndIndex(0, this.pageSize);
    this._dataSource.length.subscribe(x => this.length = x);
  }

  get dataSource() {
    return this._dataSource;
  }

  length: number;
  pageSize = 5;
  pageSizeOptions: number[] = [5, 10, 25, 100];

  onPageChange(event: PageEvent) {
    this.dataSource.setPageSizeAndIndex(event.pageIndex, event.pageSize);
  }
}
```
Here we are giving the paginator some default starting values and sending the **pageIndex** and **pageSize** to implementation of `setPageSizeAndIndex` method in the service that we will be make next.

`person-datasource.service.ts`
```typescript
export interface Person {
  name: string;
  age: number;
}

export interface Paging {
  skip: number;
  take: number;
}

@Injectable({
  providedIn: 'root'
})
export class PeopleDataSource extends DataSourceImpl<Person> {

  // local test
  people: Person[] = [
    { name: 'Person 1', age: 20 },
    { name: 'Person 2', age: 21 },
    { name: 'Person 3', age: 22 },
    { name: 'Person 4', age: 23 },
    { name: 'Person 5', age: 24 },
    { name: 'Person 6', age: 25 },
  ];

  constructor(
    // import here you https service
  ) {
    super();
  }

  async setPageSizeAndIndex(pageIndex: number, pageSize: number) {
    const startIndex = pageIndex * pageSize;
    let endIndex = startIndex + pageSize;
    if (endIndex > this.people.length) {
      endIndex = this.people.length;
    }

    const slicedPeople = this.people.slice(startIndex, endIndex);
    this.lengthBS.next(this.people.length);
    this.itemsBS.next(slicedPeople);

    // uncomment to use with backend service
    // const paging: Paging = {
    //   skip: pageIndex * pageSize,
    //   take: pageSize
    // };

    // const temp = await this.peopleService.getPeople(paging);
    // this.lengthBS.next(temp.total);
    // this.itemsBS.next(temp.items);
  }

}
```
Here you can see I have added some comments in case you would like to call a backend endpoint, but for this tutorial I will be using an local array **people** that I made and we are going to slice from it on new page index and size from the paginator.

With that our of the way let's take a look at component where we are gonna use the paginator we just made and the service.

`app.component.ts`
```typescript
export class AppComponent implements AfterViewInit  {
  dataSource: DataSourceBase;
  people: Person[] = [];

  constructor(
    private peopleDataSource: PeopleDataSource,
    private cdr: ChangeDetectorRef) {
    this.dataSource = peopleDataSource;
  }

  ngAfterViewInit(): void {
    this.peopleDataSource.itemsBS.subscribe(people => {
      this.people = people;
      this.cdr.detectChanges();
    });
  }
}
```
Here we are using **AfterViewInit** instead of **OnInit** and `this.cdr.detectChanges()` to avoid *Expression ... has changed after it was checked error*, you should be able to go with the usual **ngOnInit** if not in the main app component.

`app.component.html`
```html
<div *ngFor="let item of people">
  {% raw %}{{ item.name }} {{ item.age }}{% endraw %}
</div>
<app-base-paginator [dataSource]="dataSource"></app-base-paginator>
```
And that's about it, with this approach you will be able to use the base paginator component as many times as you like as long as you provide it with own implementation of `person-datasource.service.ts` where it will call the server for the new items for example.


