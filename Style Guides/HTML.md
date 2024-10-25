# Style Guide for HTML files

## Basic rules

- Don’t capitalize tags, including the doctype.
- Use soft tabs with two spaces—they’re the only way to guarantee code renders the same in any environment.
- Nested elements should be indented once (two spaces).
- Always use double quotes, never single quotes, on attributes.
- Don’t include a trailing slash in self-closing elements—the HTML5 spec says they’re optional.
- Don’t omit optional closing tags (e.g. </li> or </body>).
- Authors are encouraged to specify a lang attribute on the root html element, giving the document’s language. This aids speech synthesis tools to determine what pronunciations to use, translation tools to determine what rules to use, and so forth. [List of language codes](https://www.iana.org/assignments/language-subtag-registry/language-subtag-registry).
- Ensure proper content rendering by declaring an explicit character encoding. UTF-8 is the recommended encoding.
- When including CSS and JS code, there's no need to specify a _type_ attribute.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">

    <!-- External CSS -->
    <link rel="stylesheet" href="code-guide.css">

    <title>Page title</title>
  </head>

  <body>
    <img src="images/company-logo.png" alt="Company">

    <h1 class="hello-world">Hello, world!</h1>

    <!-- JavaScript -->
    <script src="code-guide.js"></script>
  </body>

  <!-- In-document CSS -->
  <style>
    /* ... */
  </style>
</html>
```

## Attribute order

HTML attributes should come in this particular order for easier reading of code.

1. `class`
2. `id`, `name`
3. `data-*`
4. `src`, `for`, `type`, `href`, `value`
5. `title`, `alt`
6. `role`, `aria-*`
7. `tabindex`
8. `style`

Attributes that are most commonly used for identifying elements should come first—class, id, name, and data attributes. Miscellaneous attributes unique to specific elements come second, followed by accessibility and style related attributes.

## What not to do

- You should try to maintain HTML standards and semantics, but not at the expense of practicallity. That means that, if there's a tag or an attribute premade in HTML that already does what you want to implement, then don't implement it, use HTML functonalities first.

```html
<!-- Good, yes, pure, demure -->
<button>...</button>

<!-- Not good, big nono -->
<div class="btn" onClick="...">...</div>
```

- You should alse try to use as little markup (tags) as possible. Don't add parent tags that don't need to be there.

```html
<!-- Not so great -->
<span class="avatar">
  <img src="...">
</span>

<!-- Better -->
<img class="avatar" src="...">
```