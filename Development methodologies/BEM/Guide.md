# BEM (Block Element Modifier)

This methodology only works using classes, and divides all possible classes between blocks, elements or modifiers.

## Block
Encapsulates a standalone entity that is meaningful on its own. While blocks can be nested and interact with each other, semantically they remain equal; there is no precedence or hierarchy.

Block names may consist of lowecase Latin letters, digits, and dashes. To form a CSS class, add a short prefix for namespacing: `.block`. Spaces in long block names are replaced by dash: `.block-long-name`.

Any DOM node can be a block if it accepts a class name: `<div class="block">...</div>`.

Use class name selector only. No tag name or ids. No dependency on other blocks/elements on a page: `.block { color: #042; }`.

## Element
Parts of a block and have no standalone meaning. Any element is semantically tied to its block.

Element names may consist of lowecase Latin letters, digits, dashes and underscores. CSS class is formed as block name plus two underscores plus element name: `.block__elem`. Spaces in long element names are replaced by dash: `.block__elem-long-name`.

```css
// Good
.block__elem { color: #042; }

// Bad
.block .block__elem { color: #042; }

div.block__elem { color: #042; }
```

## Modifier
Flags on blocks or elements. Use them to change appearance, behavior or state.

Modifier names may consist of lowecase Latin letters, digits, dashes and underscores. CSS class is formed as block’s or element’s name plus two dashes: `.block--mod` or `.block__elem--mod`. Spaces in complicated modifiers are replaced by dash: `.block--color-black`.

Modifier is an extra class name which you add to a block/element DOM node. Add modifier classes only to blocks/elements they modify, **and keep the original class**.

Use modifier class name as selector: `.block--hidden { }` To alter elements based on a block-level modifier: `.block--mod .block__elem { }` Element modifier: `.block__elem--mod { }`.

```html
<!-- Good -->
<div class="block block--mod">...</div>

<div class="block block--size-big block--shadow-yes">...</div>

<!-- Bad -->
<div class="block--mod">...</div>
```

## Nesting
**Source**: [BEM Grandchildren: How To Handle Deeply Nested Elements by Tom Ray.](https://scalablecss.com/bem-nesting-grandchild-elements/)

When nesting, things can get messy very quickly if you follow the *Block__Element__Element__Element* choice.

```html
<div class=“nav”>
    <ul class=“nav__menu”>
        <li class=“nav__menu__item”>
            <a class=“nav__menu__item__link”>Home</a>
        </li> 
    </ul>
</div>
```

For example, what happens if I want to wrap the list inside of a container?

```html
<div class=“nav”>
    <div class="nav__wrapper"> <!-- Here is my new div-->
        <!-- Now I need to refactor all the classes below
        because of the new div -->
        <ul class=“nav__wrapper__menu”>
            <li class=“nav__wrapper__menu__item”>
                <a class=“nav__wrapper____menu__item__link”>Home</a>
            </li> 
        </ul>
    <div>
</div>
```

That's why we take the grandchild solution:

```html
<div class=“nav”>
    <!-- New div added without the need to refactor -->
    <div class="nav__wrapper"> 
        <ul class=“nav__menu”>
            <li class=“nav__item”>
                <a class=“nav__link”>Home</a>
            </li> 
        </ul>
    </div>
</div>
```

## Real example

This example is from the attatched Repo, and also uses SCSS, but in a very basic form, so I'll just explain the SCSS tools used in the example quickly for everyone who doesn't know SCSS.

### Nesting
```scss
.class-a {
  // Some styles

  .class-b {
    // Some other styles
  }
}

//Is the same as
.class-a {
  // Some styles
}

.class-a .class-b {
  // Some other styles
}
```
### The _&_ operator
```scss
.card {
  // Some styles

  &__color-rectangle {
    // Some other styles
  }
}

//Is the same as
.card {
  // Some styles
}

.card__color-rectangle {
  // Some other styles
}
```

### Example from the repo
First, we have our HTML structure, where we have a father `<section class="card">` with a child `<div class="card__color-rectangle card__color-rectangle--beach">` and some other containers:

```html
<section class="card card--beach">
    <div class="card__color-rectangle card__color-rectangle--beach">
      <div class="card__title-icon-container">
          <i class="fa-solid fa-umbrella-beach card__icon"></i>
          <h2 class="heading card__title">Abades Town and Beach</h2>
      </div>

      <i class="fa-solid fa-angle-down card__down-arrow"></i>
    </div>

    <div class="card__overlay">
      <p class="card__info">
          A very long paragraph...
      </p>

    <small class="attribution card__author">Picture by Jaime Simeón Palomar Blumenthal.</small>
  </div>
</section>
```

To style this, we have the following SCSS rules:

```scss
.card {
  position: relative;
  overflow: hidden;

  // ...

  // .card__color-rectangle
  &__color-rectangle {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 5;

    // ...

    // .card__color-rectangle--beach
    &--beach { background-color: $yellow; }
  }

  // .card__title-icon-container
  &__title-icon-container {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  // .card__title, .card__icon, .card__down-arrow
  &__title,
  &__icon,
  &__down-arrow {
    color: $main-font-color;

    // ...

    // .card__title--contrast, .card__icon--contrast, .card__down-arrow--contrast
    &--contrast {
      color: $yellow;
    }
  }

  // .card__icon
  &__icon {
    margin-top: 5%;

    // ...
  }

  // .card__title
  &__title {
    margin-bottom: 5%;
    margin-inline: 10%;

    // ...
  }

  // .card__down-arrow
  &__down-arrow {
    font-size: 1.5rem;

    // ...
  }

  // .card--beach
  &--beach { background-image: url(../images/beach.png); }

  // .card__overlay
  &__overlay {
    position: absolute;
    bottom: 0;
    left: 0;

    // ...
  }

  // .card__info
  &__info {
    text-align: justify;
    color: $blue;
  }

  // .card__author
  &__author {
    // ...
  }
}
```
