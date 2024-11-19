# SCSS and SASS

CSS is the styling language that uses to style webpages. SCSS is a special type of file for SASS, a program written in Ruby that assembles CSS style sheets for a browser. SASS adds lots of additional functionality to CSS like variables, nesting and more which can make writing CSS easier and faster. SCSS files are processed by the server running a web app to output a traditional CSS that your browser can understand.

In this guide, we might use SCSS and SASS as synonyms, while we talk about their functionalities when it comes to writing code, even though SASS needs a preprocesor to convert SCSS code into CSS code the browser can read. In fact, you can run `sass input.scss output.css` to compile your SCSS code into plain CSS.

## Variables
Declaring a variable is very simple, as well as calling it.

```scss
$background-color: #fff;
$text-color: #000;

div { backgounr-color: $background-color; }

div > p { color: $text-color; }
```

## Nesting
Nesting and element inside of another means that the child element in the SCSS code is also a child element in the HTML code. It's something that could already be easily done with plain CSS, but nesting helps keeping everything organized.

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

## Partiales and imports
A partial, as its name implies, is just a part of your whole SCSS styles, but separates into a different file. These files' names are named with a leading underscore: `_whatever.scss`.

But why do that? Well, in fact, a good practice when using SCSS is to keep every view/page from your web app separated in a partial. Then, you just need to import every partial into a main SCSS file, that you link in your HTML.

For example, let's say that you have a website with a *home.html* page and a *contacts.html* page. Then you would have *layouts/_home.scss* and *layouts/_contacts.scss* partials to style both of them. Finally, you would have a *main.scss* file that you would link to every page.

```scss
// main.scss

@use "layouts/home";
@use "layouts/contacts;
```

**Note:** SASS already knows that files starting with *_* are partials, so you just need to write the name of the partial, not the name of the file.

You could also make a partial where you store all important variables and use them in a different file.

### `@use` and `@import`
Right now, the use of `@import` is deprecated in favour of `@use`.

When you import a partial, it's like if you would copy the contents of that partial into the file you are importing. This can cause conflicts if there were variables with the same name in both files, or SCSS rules that can collide.

```scss
// _colors.scss
$primary-color: #3498db;
$secondary-color: #fff

body {
  color: $secondary-color;
}

// main.scss
@import 'colors';
body {
  color: $primary-color;
}

/* Which text color is the body using? */
```

In this example, the body's text color will be the last one delcared (`#3498db`). So there is a difference if we import the partial before or after the rule:

```scss
// main.scss
body {
  color: $primary-color;
}

@import 'colors';
```

Now, the body's text color will be `#fff`.

When you use a partial, it's like importing one, but in a different namespace. That way, all the contents of the partial are separated.

```scss
// _colors.scss
$primary-color: #3498db;
$secondary-color: #fff

body {
  color: $secondary-color;
}

// main.scss
@use 'colors';
body {
  color: colors.$primary-color;
}
```

Now, it doesn't matter where we use the partial (even though SASS makes us write the `@use` statements first in the files), the body's text color will allways be `#3498db`. Also, when using variables form a partial we need to specify the namespace, which will be the name of the partial: `_colors.scss` will have `colors` as a namespace when using it.

## Mixins and functions
A mixin is a function that returns a chunk of code.

```scss
@mixin theme($theme: false) {
  @if $theme == false {
    background-color: #000;
  } @else {
    @content;
  }
}

.block {
  @include theme(true) {
    background-color: #fff;
  }
}
```

This mixin is pretty useles, but is usefull to explain the concept. `theme()` has a parameter, which is *false* by default. If we don't set it, it returns a preseted CSS rule, but if we specify that `$theme` is *true*, it returns the rules we want.

SASS also have prebuilt functions from different libraries, like math functions:

```scss
@use "sass:math";

article {
  width: math.div(600px, 960px) * 100%;
}
```

## Using SCSS to personalize Bootstrap classes
For example, let's change the theme colors that Bootstrap uses when we add classes such as `btn-primary, bg-light`, etc.

To do that, we just need to go to our variables partial and override the theme colors map:

```scss
$theme-colors: (
  "primary":    #fff,
  "secondary":  #fabada,
  "success":    #121212,
  "info":       #000,
  // ...
);
```
