# Observables (rxjs module)ç

Observables are a tool to work with sequences of asynchronous data.

![Graph](image.png)

## Observables VS Promises
Observables and Promises are both used to handle asynchronous operations, but they differ significantly in terms of functionality, flexibility, and usage patterns.

Pormises return a single value (or an error) once the operation completes. Once a promise is resolved or rejected, it cannot be reused. They start the operation immediately upon creation. You can't "subscribe" to a promise multiple times; it will resolve only once.

Meanwhile, Observables can emit multiple values over time, meaning it can produce a stream of values and handle events in real-time. They're also *lazy*. They won’t start producing values until there's a subscription to it. You can create multiple subscriptions, and each one will trigger the observable to start executing.

The following example will show a function from a service created to manage posts from an API. You can access the full project inside of the *Repo* folder.

```ts
// In /services/post.service.ts
// Declaration of updatePost() as a Promise
updatePost(postId: string, post: PostDTO): Promise<PostDTO> {
  return this.http.put<PostDTO>(this.urlBlogUocApi + '/' + postId, post).toPromise();
}

// Declaration of updatePost() as an Observable
updatePost(postId: string, post: PostDTO): Observable<PostDTO> {
    return this.http.put<PostDTO>(this.urlBlogUocApi + '/' + postId, post).pipe(
      catchError((error) => {
          this.sharedService.errorLog(error.error);

          return of(new PostDTO('', '', 0, 0, new Date()));
      })
    );
  }
```

```ts
// In /components/posts/post-form/post-form.component.ts
// Call to updatePost() as a Promise
try {
    await this.postService.updatePost(this.postId, this.post);

    responseOK = true;
} catch (error: any) {
    errorResponse = error.error;
    this.sharedService.errorLog(errorResponse);
}

// Call to updatePost() as an Observable
this.postService.updatePost(this.postId, this.post).subscribe({
  complete: () => {
      responseOK = true;
  }
});
```
Notice how, we changed the error handling from the calling to the declaring of the function. We can also handle errors when calling the function, but it's considered best practice not doing it.

## Error handling
There are two ways to do it:

1. When declaring the observable. **This is a best practice**:

```ts
// In /services/post.service.ts

likePost(postId: string): Observable<updateResponse> {
  return this.http.put<updateResponse>(this.urlBlogUocApi + '/like/' + postId, NONE_TYPE)
    .pipe(catchError((error) => {
      this.sharedService.errorLog(error.error);

      return of({ affected: -1 } as updateResponse);
    })
  );
}
```

In this case, after handling the error, we must return the same type as the observable. In this case, *of* creates an observable with the value and type we input as a parameter.

2. When running (subscribing) to the Observable:

```ts
// This isn't on any file of the Repo folder.
// It's juts an example

this.postService.likePost(postId).subscribe({
  error: (error) => {
    this.sharedService.errorLog(error.error);
  }
});

// Alternative way
this.postService.likePost(postId)
  .pipe(catchError((error) => {
    this.sharedService.errorLog(error.error);

    return of({ affected: -1 } as updateResponse);
  })
  .subscribe();
```

## `complete()` VS. `next()`
When running asynchronous code, we may want to run synchronous code only after the asynchronous one. And when treating with Observables, which are collections of asynchronous data, we might want to run synchronous code after every chung of data to process every item in a certain way. This is why we use `complete()` and `next()`.

Let's say that, in our project, we have a component with an attribute that holds all posts that come from an API, and a function that subscribes to an Observable that makes ths HTTP request to the API and returns those posts.

```ts
// In Services/post.service.ts
// This would be the service that makes the HTTP request to the API
getPosts(): Observable<PostDTO[]> {
  return this.http.get<PostDTO[]>(this.urlBlogUocApi).pipe(
    catchError((error) => {
        this.sharedService.errorLog(error.error);

        return of([new PostDTO('', '', 0, 0, new Date())]);
    })
  );
}
```

```ts
// In Components/home/home.component.ts
export class HomeComponent {
  posts!: PostDTO[];
  
  // Other unimportant attributes

  showButtons: boolean;
  constructor(
    private postService: PostService,
    // Some other injections
  ) {
    // Some constructor actions

    // Call to function that calls the PostService
    this.loadPosts();
  }

  private async loadPosts(): Promise<void> {
    // Some logic

    this.postService.getPosts().subscribe({
      next: (postResult) => {
        this.posts = postResult;
      },
      complete: () => {
        const tmpCategories: CategoryDTO[][] = [...this.posts.map(post => post.categories)];
        let allCategoriesUID: string[] = [];

        tmpCategories.forEach((array: CategoryDTO[]) => {
          array.forEach((category: CategoryDTO) => {
            if (!allCategoriesUID.includes(category.categoryId)) {
              this.allCategories.push(category);
              allCategoriesUID.push(category.categoryId);
            }
          })
        })
      }
    });
  }
}
```

We process the data in the *next()* block every time we get new information, and do everything else in the *complete()* block, only after the Observable is done emiting information.

* `next()`: This function runs each time the Observable emits a new value (or item). It’s called for every emission, so if the Observable emits multiple values over time, `next()` will execute for each of them. It’s ideal for processing or handling each individual data item as it arrives.

* `complete()`: This function runs only once, when the Observable has finished emitting all values and completes its sequence. It signifies the end of the Observable’s lifecycle, meaning no more values will be emitted, and no further calls to `next()` will occur. The `complete()` function is ideal for cleanup actions or final steps that should only happen after all data is received and the Observable has stopped.

## Basic operators

* **from**: Makes an Observable out of and array or a Promise.
```ts
const arraySource = from([1, 2, 3, 4]);

// output: "1", "2", "3", "4"
const subscribe = arraySource.subscribe({
  next: (val) => { console.log(val); }
});
```

* **of**: Makes and Observable out of a sequence.
```ts
const sequenceSource = of(1, 2, 3, 4);

// output: "1", "2", "3", "4"
const subscribe = sequenceSource.subscribe({
  next: (val) => { console.log(val); }
});
```

* **interval**: Emits consecutive numbers in between specified intervals.
```ts
const intervalSource = interval(1000); // in ms

// output: "0", waits 1s, "1", waits 1s, ...
const subscribe = intervalSource.subscribe({
  next: (val) => { console.log(val); }
});
```

* **timer**: Has two possible behaviours. If you pass only one argument, it waits said amount (in ms) to emit. If you pass two, it waits the first argument in ms to emit, and keeps emiting every second argument in ms.
```ts
const sourceTimer1_OnlyEmitsOnce = timer(1000);
const sourceTimer2_EmitsEvery4s = timer(1000, 4000);

// output: waits 1s, "0", ends
const subscribe = sourceTimer1_OnlyEmitsOnce.subscribe({
  next: (val) => { console.log(val); }
});

// output: waits 1s, "0", waits 4s, "1", waits 4s, "2", ...
const subscribe = sourceTimer2_EmitsEvery4s.subscribe({
  next: (val) => { console.log(val); }
});
```

## Transformation operators

* **map**: Applies a tranformation to every item of the observable.

```ts
const arraySource = from([1, 2, 3, 4]);

// output: "2", "4", "6", "8"
const subscribe = arraySource
  .pipe(map(val => val * 2))
  .subscribe({
    next: (val) => { console.log(val); }
  });
```

* **reduce**: Adds all values.
```ts
const arraySource = from([1, 2, 3, 4]);

// output: "10"
const subscribe = arraySource
  .pipe(reduce((acc, val) => acc + val))
  .subscribe({
    next: (val) => { console.log(val); }
  });
```

## Filter operators

* **filter**: Emits only the values that match the specified requisite.
```ts
const arraySource = from([1, 2, 3, 4]);

// output: "2", "4"
const subscribe = arraySource
  .pipe(filter(num => num % 2 === 0))
  .subscribe({
    next: (val) => { console.log(val); }
  });
```

* **debounceTime**: Filters out the items emitted before a specified time interval.

```ts
// Emit the most recent click after a burst of clicks
const clicks = fromEvent(document, 'click');
const result = clicks.pipe(debounceTime(1000));
result.subscribe(x => console.log(x));
```

* **take**: Limits the amount of emitted items.
```ts
const arraySource = from([1, 2, 3, 4]);

// output: "1", "2"
const subscribe = arraySource
  .pipe(take(2))
  .subscribe({
    next: (val) => { console.log(val); }
  });
```

* **takeUntil**: Emits values until the specified source starts emiting values.
```ts
// Emits every 1s
const intervalSource = interval(1000);

// Emits once after 4s
const timerStopper = timer(4000);

// When timerStopper emits after 4s, stop emitting intervalSource
// output: "0", "1", "2", "3", stop 
const subscribe = intervalSource
  .pipe(takeUntil(timerStopper))
  .subscribe({
    next: (val) => { console.log(val); }
  });
```

## Combine operators

* **combineLatest**: Combines multiple Observables to create an Observable whose values are calculated from the latest values of each of its input Observables.
```ts
import { timer, combineLatest } from 'rxjs';

const firstTimer = timer(0, 1000); // emit 0, 1, 2... after every second, starting from now
const secondTimer = timer(500, 1000); // emit 0, 1, 2... after every second, starting 0,5s from now
const combinedTimers = combineLatest([firstTimer, secondTimer]);
combinedTimers.subscribe(value => console.log(value));
// output
// [0, 0] after 0.5s
// [1, 0] after 1s
// [1, 1] after 1.5s
// [2, 1] after 2s
```

* **concat**: Creates an output Observable which sequentially emits all values from the first given Observable and then moves on to the next.
```ts
const source1 = of(1, 2, 3);
const source2 = of(4, 5, 6);

// output: "1", "2", "3", "4", "5", "6"
const subscribe = source1
  .pipe(concat(source2))
  .subscribe({
    next: (val) => { console.log(val); }
  });
```

* **merge**: Combines outputs from different Observables into one stream.
```ts
//emit every 2.5 seconds
const first = interval(2500);
//emit every 2 seconds
const second = interval(2000);
//emit every 1.5 seconds
const third = interval(1500);
//emit every 1 second
const fourth = interval(1000);

//emit outputs from one observable
const example = merge(
  first.pipe(mapTo('FIRST!')),
  second.pipe(mapTo('SECOND!')),
  third.pipe(mapTo('THIRD')),
  fourth.pipe(mapTo('FOURTH'))
);
//output: "FOURTH", "THIRD", "SECOND!", "FOURTH", "FIRST!", "THIRD", "FOURTH"
const subscribe = example.subscribe(val => console.log(val));
```

## Other operators

* **toPromise**: Transforms an Observable into a Promise.
```ts
const observable = of('This is a Promise');

// output: "This is a Promise"
const promise = observable.toPromise()
  .then((val) => {
    console.log(val);
  });
```

But this would make much more sense if we made a function out of an observable, so we could input values to the promise, make an array and run all promises with Promise.all().
```ts
//return basic observable
const sample = val => Rx.Observable.of(val).delay(5000);
/*
  convert each to promise and use Promise.all
  to wait for all to resolve
*/
const example = () => {
  return Promise.all([
    sample('Promise 1').toPromise(),
    sample('Promise 2').toPromise()
  ]);
};
//output: ["Promise 1", "Promise 2"]
example().then(val => {
  console.log('Promise.all Result:', val);
});
```

