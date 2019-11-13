---
layout: post
title:  "Update collection based on second collection"
date:   2019-11-13 18:50:32 +0200
categories: typescript
author: Maverick
comments: true
---

In this tutorial we are going to edit the first collection or array based on the content of the first one, what I mean by that is if you have two items in the second collection and one exist and other does not exist in the first collection, we would like to remove the one that exist and add this one that does not exists.

[Finished Live Example](https://stackblitz.com/edit/update-collection-based-on-second-collection)

As it is typescript I made a simple model

```typescript
export interface IPerson {
  id: number;
  name: string;
}
```

`first collection`
```typescript
const people: IPerson[] = [
  {
    id: 1,
    name: 'Maverick',
  },
  {
    id: 2,
    name: 'Arxero',
  },
  {
    id: 3,
    name: 'Feruchio',
  },
];
```

`second collection`
```typescript
const peoplesEdit: IPerson[] = [
  {
    id: 1,
    name: 'Maverick',
  },
  {
    id: 6,
    name: 'asd',
  },
];
```
As I said in the beginning, if an element from the **second collection** exist in the **first** then that one should be **removed** from the first and if it does not exist it should be **added**.

```typescript
for (const person of peoplesEdit) {
  if (!peoplesClone.find(x => x.id === person.id)) {
    people.push(person);
  }
}

for (const person of peoplesClone) {
  if (peoplesEdit.find(x => x.id === person.id)) {
    people.splice(people.indexOf(person), 1);
  }
}
```
This is exactly what we are doing here, we are traversing in both directions and checking if the current element exists or does not exist depending on the traverse and then we add or remove the element.