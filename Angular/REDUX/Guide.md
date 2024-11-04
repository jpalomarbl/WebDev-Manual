# REDUX
Redux (typically implemented with NgRx) is a state management pattern that centralizes application state and makes it predictable by organizing it in a single, immutable data store. Redux in Angular uses actions (describing events), reducers (pure functions that update the state based on actions), and selectors (queries to read specific parts of the state). This setup makes it easier to manage and debug complex state changes, especially in large applications, by ensuring that the app's state is updated in a consistent, traceable way. NgRx integrates Redux principles directly with Angular's reactive programming model, making it highly compatible with Angular applications.

## Basic concepts
* **Store**: The Store is a centralized, immutable state container that holds the entire application state. It acts as a single source of truth, where all components can access and update data consistently.
* **Actions**: Actions are dispatched events that describe a change or event happening in the application (e.g., loading data, updating a user). Each action has a type (a unique identifier) and can carry a payload with additional data for the event.
* **Reducers**: Reducers are pure functions that take the current state and an action as inputs, and return a new state based on the action’s type and payload. They define how the application state changes in response to actions.
* **Selectors**: Selectors are functions used to retrieve specific data from the Store, optimizing performance by minimizing unnecessary state recalculations and helping manage complex data structures.

![Redux diagram](img/image.png)

In short, when we want to make a change to the application's data, we subscribe to the **store** and we dispatch an **action**. By doing that, we are calling the **reducer**, which combines the current state of the **store** with the **action** we dispatched to make the change to the state of the **store**.

## Installation and configuration
After creating our Angular project, we need to install `ngrx/store`. So we run `npm install @ngrx/store --save`.

Now, for this example, we will be implementing a counter app. This app will have a counter and three components:

* **Mother component**: This component will show the counter and will have a button to **increase** (+1) and to **decrease** (-1) it.
* **Daughter component**: This one will also display the counter's value, and will have a **duplicate** (*2) button.

Inside of the Repo folder you will have an implementation of the app with and without REDUX so you can compare.

1. We need to create all the different actions, so we create a `app/Counter/counter.actions.ts` file. Inside of it, we define all the actions **just by name. We don't need to implement what they do here.**

```ts
// In app/Counter/counter.actions.ts
import { createAction, props } from "@ngrx/store";

export const increment = createAction('[Counter Component] Increment');
export const decrement = createAction('[Counter Component] Decrement');
export const duplication = createAction('[Counter Component] Duplication');
```

2. Now we need to create the reducer, so we make a `app/Counter/counter.reducer.ts` file. Inside of it, we will have a function that's basically a switch case conditional selector that takes the current state of te **store** and an action as arguments.

```ts
// In app/Counter/counter.reducer.ts

import { Action, createReducer, on } from "@ngrx/store";
import * as actions from "./counter.actions"; // We import all actions into an object called "actions"

// We set the initial state of the counter
export const initialState = 20;

// We call a reducer constructor that takes the initial state
// Then decides what to do with it.
const _counterReducer = createReducer(
  initialState,
  on(actions.increment, (state) => state + 1),  // Increment adds 1
  on(actions.decrement, (state) => state - 1),  // Decremend substracts 1
  on(actions.duplication, (state) => state * 2)  // Duplicate duplicates
);

// We export a function that takes the state and an action
export function counterReducer(state: number | undefined, action: Action) {
  return _counterReducer(state, action);
}
```

This is the same as doing the following:

```ts
export function counterReducer(state: number = 20, action: Action) {
  switch (action.type) {
    case increment.type:
      return state + 1;

    case decrement.type:
      return state - 1;

    case duplication.type:
      return state * 2;

    default:
      return state;
  }
}
```

3. Next, we need to add the reducer to the store.

```ts
// In app.config.ts

// Other imports
// ...
import { provideStore } from '@ngrx/store';
import { counterReducer } from './Counter/counter.reducer';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideClientHydration(),
    provideStore({ counter: counterReducer }) // We add the reducer, that sets the "counter" property form the store
  ]
};
```

5. Next, for simplicity, we need to create an interface that represents a state of the store. In this case it will be easy, as the store only holds one attribute: counter. We need to create a `app/app.reducer.ts` file.

```ts
// In app/app.reducer.ts
export interface AppState {
  counter: number;
}
```

6. Now, let's create our mother component.

```ts
// In app.component.ts
import { Store } from '@ngrx/store';
import * as actions from "./Counter/counter.actions";
import { AppState } from './app.reducer';
// Other imports
// ...

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    RouterOutlet
  ],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  title = 'counter-app-with-redux';
  // The attribute that will hold the counter value inside of this component
  counter!: number;

  constructor(private store: Store<AppState>) {
    // In larger apps, we would have a ton of properties,
    // so we select only the ones that we want and subscribe to them.
    this.store.select('counter').subscribe((counter) => {
      this.counter = counter;
    });
  }

  increase(): void {
    // By calling dispatch and passing an action to it
    // REDUX by itself calls the reducer and passes the action and the current state.
    this.store.dispatch(actions.increment());
  }

  decrease(): void {
    this.store.dispatch(actions.decrement());
  }
}
```

This would be the HTML structure. We are using bootstrap for the styles, but feel free to add your own:

```html
<!-- In app.component.html -->

<div class="jumbotron d-flex flex-column justify-content-center align-items-center min-vh-100">
  <div class="container text-center">
      <div class="row" style="text-align: center">
          <div class="col">
              <h1> Counter (Mother) </h1>
              <h1>{{ counter }}</h1>
          </div>
      </div>
  </div>

  <div class="row" style="text-align: center;">
    <div class="col">
      <button
        (click)="increase()"
        type="button"
        class="btn btn-primary btn-lg"
      >
        Increase (+1)
      </button>
    </div>
  </div>

  <div class="row mt-2" style="text-align: center;">
    <div class="col">
      <button
        (click)="decrease()"
        type="button"
        class="btn btn-success btn-lg"
      >
        Decrease (-1)
      </button>
    </div>
  </div>
</div>
```

It should look like this:

![Mother component](img/image-1.png)

If we press the buttons, it adds and subtracts from the counter.

7. Now, let's create our child component. To do that, we will run `ng g c Counter/daughter`. This will create a new folder inside of our Counter directory for the new component.

```ts
// In daughter.component.ts

import { Store } from '@ngrx/store';
import * as actions from '../counter.actions';
import { AppState } from '../../app.reducer';
// Other imports
// ...

@Component({
  selector: 'app-daughter',
  standalone: true,
  imports: [],
  templateUrl: './daughter.component.html',
  styleUrl: './daughter.component.scss'
})
export class DaughterComponent {
  // The attribute that will hold the counter value inside of this component
  counter!: number;

  constructor(private store: Store<AppState>) {
    // In larger apps, we would have a ton of properties,
    // so we select only the ones that we want and subscribe to them.
    this.store
      .select('counter')
      .subscribe((counter) => { this.counter = counter; });
  }

  duplicate(): void {
    // By calling dispatch and passing an action to it
    // REDUX by itself calls the reducer and passes the action and the current state.
    this.store.dispatch(actions.duplication());
  }
}
```

This is the HTML sctructure for the daughter component.

```html
<!-- In daughter.component.html -->
<div class="row text-center d-flex flex-column justify-content-center align-items-center">
  <div class="col">
    <h1> Counter (daughter) </h1>
    <h1>{{ counter }}</h1>
  </div>

  <div class="row mt-2" style="text-align: center;">
    <div class="col">
      <button (click)="duplicate()" type="button" class="btn btn-warning btn-lg">
        Duplicate (*2)
      </button>
    </div>
  </div>
</div>
```

Now, let's insert our daughter component into our mother component.

```html
<!-- In app.component.html -->
<div class="jumbotron d-flex flex-column justify-content-center align-items-center min-vh-100">
  <!-- All the HTML code from the previous step -->

  <hr class="w-100 h-1"/>
  <hr class="w-100 h-1"/>

  <app-daughter></app-daughter>
</div>
```

Now, it should look like this:

![Daughter component](img/image-2.png)

And that's it. We have an app with several components that all modify the same data, but don't need to comunicate between each other, because everything is stored in the **store**.

## Props
We can add props to actions, that act like arguments of a function. For example, let's add a prop to our *duplicate* action so we have to pass a **2** as a prop when we call it.

1. First let's modify our action.
```ts
// In counter.actions.ts

// export const duplication = createAction('[Counter Component] Duplication');
export const duplication = createAction('[Counter Component] Duplication', props<{ number: number }>());
```

**VERY IMPORTANT**: *props* is a funtion, so you need to add the parenthesis at the end.

2. Now, we need to change the reducer so it multiplies the state by *number*

```ts
// In counter.reducer.ts
const _counterReducer = createReducer(
  initialState,
  on(actions.increment, (state) => state + 1),
  on(actions.decrement, (state) => state - 1),
  on(actions.duplication, (state, { number }) => state * number) // Here
);
```

3. Finally, we just need to call it properly in our daughter component:

```ts
// In daughter.component.ts
duplicate(): void {
  this.store.dispatch(actions.duplication({ number: 2 }));
}
```
