# Lazy Loading
[Angular's official guide on Lazy Loading.](https://v17.angular.io/guide/lazy-loading-ngmodules)

In Angular, Lazy Loading refers to a specific routing set of tools.

Let's say that you have an app with three pages. Without Lazy Loading, when you enter the home page, Angular also loads all other pages and components from the app, which can be very harmful to the app's performance.

If we implement Lazy Loading, the pages and components will only load when the user visits the specific route they're in.

## How to implement Lazy Loading

1. First, we need to set up the application by running `ng new [APP NAME] --no-standalone`.

This will set up a normal Angular application, with a special `app-routing.module.ts` file we will configure later.

2. Next, we will create our component. We will set up a Home-page Component as a module by running `ng generate module home-page --route home --module app.module`.

This is going to create a *home-page* component, with the following structure:

![Home-page-component structure](image.png)

As you can see, it created 6 files:
* A routing file.
* A CSS styles file.
* An HTML View file.
* A tests file.
* A TS Script file.
* A module file.

Let's focus on the module file:

```ts
// In home-page.module.ts

import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';

import { HomePageRoutingModule } from './home-page-routing.module';
import { HomePageComponent } from './home-page.component';


@NgModule({
  declarations: [
    HomePageComponent
  ],
  imports: [
    CommonModule,
    HomePageRoutingModule
  ]
})
export class HomePageModule { }
```

Here we declare and import every module that makes our new component work, instead of doing it inside of `home-page.component.ts` as we normally would. Here's where all the component's dependencies are managed.

Now, let's look at the routing file:

```ts
// In home-page-routing.module.ts

import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomePageComponent } from './home-page.component';

const routes: Routes = [{ path: '', component: HomePageComponent }];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class HomePageRoutingModule { }
```

Here it's specified that when this route is accesed, the *home-page* component has to be loaded. Angular does not specify a route because that's done in the app's main routing file:

```ts
// In app-routing.module.ts

const routes: Routes = [
  { 
    path: 'home',
    loadChildren: () => import('./home-page/home-page.module').then(m => m.HomePageModule)
  }
];
```

You can see that, while in a regular Angular routing file we could directly load the component when specifying a route, here we use the `loadChildren()` function, to *lazy load* it.

We load the whole component module, and then, the module's routing file loads the component.

3. Finally, let's add a default route. This makes more sense when we have more than one route.

```ts
// In app-routing.module.ts

const routes: Routes = [
  {
    path: 'home',
    loadChildren: () =>
      import('./home-page/home-page.module').then((m) => m.HomePageModule),
  },

  {   // Default route
    path: '',
    redirectTo: 'home',
    pathMatch: 'full'
  }
];
```

This makes it so that whenever a route that's not specified is accesed, it automatically redirects the user to the *home* route.
