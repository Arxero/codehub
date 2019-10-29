---
layout: post
title:  "Using Angular’s http client interchangeable with Axios"
date:   2019-10-17 18:50:32 +0200
categories: angular
author: Maverick
comments: true
---

In this short tutorial, you are going to be learning how do you replace Angular’s default http client with Axios in case you would like to experiment.

Let’s first start with creating a new angular project and then creating a simple service, which by the way would need to use a base implementation of a generic http service.

[Finished Live Example](https://stackblitz.com/edit/angular-wylrjv)

    ng new axios-tutorial
    cd axios-tutorial
    npm i axios

With that out of the way let’s write our generic service I mentioned above. I will be making only one just the get method, but other are in the same way.

    ng g s base-http-client

```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Observable } from 'rxjs';


@Injectable({
  providedIn: 'root'
})
export class BaseHttpClientService {

  constructor(private http: HttpClient) { }

  get<T>(endPoint: string): Observable<T> {
    return this.http.get<T>(endPoint, this.SetHeaders());
  }

  private SetHeaders() {
    let headers = new HttpHeaders();
    headers = headers.append('Authorization', 'someAuthToken');
    return { headers };
  }
}
```

    ng g s github-profile

```typescript
import { Observable } from 'rxjs';
import { BaseHttpClientService } from './base-http-client.service';
import { Injectable } from '@angular/core';


export interface IGitHubProfileDto {
  login: string;
  id: string;
  avatar_url: string;
  created_at: string;
  blog: string;
  location: string;
}

@Injectable({
  providedIn: 'root'
})
export class GithubProfileService {

  constructor(private http: BaseHttpClientService) { }

  getGitHubProfile(url: string): Observable<IGitHubProfileDto> {
    return this.http.get<IGitHubProfileDto>(url);
  }
}
```

[download-axios-tutorial-before-axios]({{site.baseurl}}/assets/files/axios-tutorial-before-axios.rar)

At this point, there is nothing new, but here where we will make a base-http-client-axios service and later on we will plug it in our github-profile service (without changing anything) with the help of the angular’s dependency injection.

    ng g s base-http-client-axios

Make an interface in it, include your crud methods in it and then make both base service classes implement this interface.

```typescript
export interface HttpClientService {
  get<T>(endPoint: string): Observable<T>;
}
```

`base-http-client-axios.service.ts`

```typescript
import { Injectable } from '@angular/core';
import { Observable, from } from 'rxjs';
import axios, { AxiosRequestConfig } from 'axios';
import { map } from 'rxjs/operators';

export interface HttpClientService {
  get<T>(endPoint: string): Observable<T>;
}

@Injectable({
  providedIn: 'root'
})
export class BaseHttpClientAxiosService implements HttpClientService {

  constructor() { }


  get<T>(endPoint: string): Observable<T> {
    return from(axios.get<T>(endPoint, this.SetHeaders())).pipe(map(x => x.data));
  }

  private SetHeaders() {
    // tslint:disable-next-line: no-angle-bracket-type-assertion
    const defaultOptions =  <AxiosRequestConfig>{
      headers: {
        Authorization: 'someAuthToken',
      },
    };

    return defaultOptions;
  }
}
```

Here you can see the difference in the form of `from`, this is because axios returns a promise, but we would like to transform it into observable in order to use it in all our service without changing much. The other thing worth mentioning is using the pipe and map operators to get the result from axios which is an object with property data.

**Inject our interface in our other services like so:**

```typescript
constructor(@Inject('HttpClientService') private http: HttpClientService) { }
```

This is also called programming into interface instead of an implementation of a class, which is always recommended in OOP.

Now what is left to do is import it in `app.module.ts` in providers array.

```typescript
{
      provide: 'HttpClientService', // our interface
      useClass: BaseHttpClientAxiosService, // the base class we 
      can switch to
}
```

Now we are using `axios` client without our `github-profile service` knowing about it at all, this is pretty handy and useful in case you need it to switch to angular’s http client or axios, just change `useClass` value with angular’s one (in our case `BaseHttpClientService`).

[download-axios-tutorial-after-axios]({{site.baseurl}}/assets/files/axios-tutorial-after-axios.rar)