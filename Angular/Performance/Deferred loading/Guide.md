# Deferred loading
[Deferred loading's Angular guide.](https://angular.dev/guide/templates/defer)

Let's say that we have a page where we have a few blocks of text content, a few images, and a child component that takes about 3 seconds to load. In this scenario, by default, Angular will wait for the child component to load, to render the whole page, which doesn't make any sense. The user should be able to see all the other content while they wait for the component to load.

This is why we use `@defer` blocks:

```html
<main class="container">
  <p> This is some text </p>
  <img src="imgSrc.png/>

  @defer (on viewport) {
  <slow-app-component></slow-app-component>
  } @placeholder {
  <p>The chart is being loaded. Please hold on...</p>
  }
</main>
```

In this case, the text and image will load first, and while the `<slow-app-component>` loads, the text placeholder will show. There are various configurations, but in this case, we're using `@defer (on viewport)`, which means that the component won't start to load until it reaches the viewport.
