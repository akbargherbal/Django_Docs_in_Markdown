# JavaScript code

While most of Django core is Python, the `admin` and `gis` contrib apps
contain JavaScript code.

Please follow these coding standards when writing JavaScript code for inclusion
in Django.

## Code style

* Please conform to the indentation style dictated in the `.editorconfig`
  file. We recommend using a text editor with [EditorConfig](https://editorconfig.org/) support to avoid
  indentation and whitespace issues. Most of the JavaScript files use 4 spaces
  for indentation, but there are some exceptions.
* When naming variables, use `camelCase` instead of `underscore_case`.
  Different JavaScript files sometimes use a different code style. Please try to
  conform to the code style of each file.
* Use the [ESLint](https://eslint.org/) code linter to check your code for bugs and style errors.
  ESLint will be run when you run the JavaScript tests. We also recommended
  installing a ESLint plugin in your text editor.
* Where possible, write code that will work even if the page structure is later
  changed with JavaScript. For instance, when binding a click handler, use
  `$('body').on('click', selector, func)` instead of
  `$(selector).click(func)`. This makes it easier for projects to extend
  Django’s default behavior with JavaScript.

<a id="javascript-patches"></a>

## JavaScript patches

Django’s admin system leverages the jQuery framework to increase the
capabilities of the admin interface. In conjunction, there is an emphasis on
admin JavaScript performance and minimizing overall admin media file size.

<a id="javascript-tests"></a>

## JavaScript tests

Django’s JavaScript tests can be run in a browser or from the command line.
The tests are located in a top level [js_tests](https://github.com/django/django/blob/main/js_tests) directory.

### Writing tests

Django’s JavaScript tests use [QUnit](https://qunitjs.com/). Here is an example test module:

```javascript
QUnit.module('magicTricks', {
    beforeEach: function() {
        const $ = django.jQuery;
        $('#qunit-fixture').append('<button class="button"></button>');
    }
});

QUnit.test('removeOnClick removes button on click', function(assert) {
    const $ = django.jQuery;
    removeOnClick('.button');
    assert.equal($('.button').length, 1);
    $('.button').click();
    assert.equal($('.button').length, 0);
});

QUnit.test('copyOnClick adds button on click', function(assert) {
    const $ = django.jQuery;
    copyOnClick('.button');
    assert.equal($('.button').length, 1);
    $('.button').click();
    assert.equal($('.button').length, 2);
});
```

Please consult the `QUnit` documentation for information on the types of
[assertions supported by QUnit](https://api.qunitjs.com/assert/).

### Running tests

The JavaScript tests may be run from a web browser or from the command line.

#### Testing from a web browser

To run the tests from a web browser, open up [js_tests/tests.html](https://github.com/django/django/blob/main/js_tests/tests.html) in your
browser.

To measure code coverage when running the tests, you need to view that file
over HTTP. To view code coverage:

* Execute `python -m http.server` from the root directory (not from inside
  `js_tests`).
* Open [http://localhost:8000/js_tests/tests.html](http://localhost:8000/js_tests/tests.html) in your web browser.

#### Testing from the command line

To run the tests from the command line, you need to have [Node.js](https://nodejs.org/) installed.

After installing `Node.js`, install the JavaScript test dependencies by
running the following from the root of your Django checkout:

```console
$ npm install
```

Then run the tests with:

```console
$ npm test
```
