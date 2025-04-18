+++
title = 'CSS Attribute Selectors: ~= vs. *='
+++

Within CSS attribute selectors, the operators `~=` (the word finder) and `*=` (the substring spotter) offer distinct matching criteria:

- `~=` operator selects elements whose attribute value contains a specified **whole word** from a space-separated list.

    ```css
    [class~="button"] {...}
    ```

    You're telling the browser to select any element whose `class` attribute contains the *whole* word "button".
    It has to be a standalone word, usually separated by spaces.

    - ✅ `<div class="button primary">Click Me</div>`
    - ✅ `<a class="call-to-action button">Go!</a>`
    - ✅ `<span class="icon button"></span>`
    - ❌ `<div class="button-primary">Nope, not a whole word.</div>`
    - ❌ `<a class="submitbutton">Still not a whole word.</a>`

- `*=` operator selects elements whose attribute value contains a specified **substring** at any position.

    ```css
    [href*="example"] {...}
    ```

    You're telling the browser to select any element whose `href` attribute contains the *substring* "example" anywhere within its value.

    - ✅ `<a href="https://www.example.com">Our Example Site</a>`
    - ✅ `<a href="mailto:test@example.org">Email Us</a>`
    - ✅ `<a href="/products/example-widget.html">Check this out!</a>`
    - ✅ `<a href="another-site?ref=example123">Another Link</a>`
    - ❌ `<a href="https://www.different.com">Nope, no "example" here.</a>`

See the difference? `*=` doesn't care if "example" is a whole word or part of a larger string. It just needs to be present somewhere in the attribute's value.

## References

- [CSS Attribute Selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors)
- [CSS Selectors - Learn web development](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Attribute_selectors)


