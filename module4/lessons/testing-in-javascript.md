---
layout: page
title: Testing in Javascript
tags: Unit and Integration testing libraries
---

[//]: (For Instructors:)
[//]: (There is an old version of this lesson that uses webpack. Just peek at the Git history)

Goals
----------

-   Can write unit tests in javascript
-   Can write integration tests in javascript

Libraries covered
---------

- [Mocha](https://mochajs.org/) - A test runner
- [Chai](http://chaijs.com/) - An assertion library
- [mocha-phantomjs](https://github.com/nathanboktae/mocha-phantomjs) - A way to run Mocha tests in the headless browser, PhantomJS
- [jQuery](https://api.jquery.com/) - We'll use it for interacting with our DOM

Test Type Review
-----------

- Why test at all?
- What are the different types of tests?
- Why would we use one over the other?

Our Starter Repo
-------------

To get us started, clone down a fresh copy of the [Quantified Self Starter Kit](https://github.com/turingschool-examples/quantified-self-os). To clone it into a folder named for this lesson use this command:

```
$ git clone https://github.com/turingschool-examples/quantified-self-os.git testing-in-javascript
```

Mocha - the test runner
-------------

Mocha has one job, to run your tests. It doesn't even do assertions. We'll get to that in a minute.

Mocha gives you a few functions right off the bat. You might recognize them if you're used to using RSpec:

```js
  describe()
  context()
  it()
```

They're used similarly to RSpec. The biggest different how you pass a block to the function. Since there isn't a `do...end` in JavaScript, we pass a "callback" function:

```js
describe("Something that I'm describing", function(){
  context("That thing under some context", function(){
    it("does a thing", function(){

    });
  });
});
```

For future reference: If you're familiar with ES6 [Arrow Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions), these will make your callbacks slightly less verbose. I'm not going to bother with them for this lesson.

Save the above text in a file named `test/test.js`. Add your new test file to `test/test-template.html`. Then run:

```
$ open test/test-template.html
```

You should see something like

```
Something that I'm describing
  That thing under some context
    ✓ does a thing


1 passing (8ms)
```

As a quick note: `describe()` and `context()` are not necessary. Typically, you'll want at least a `describe()` block to tell us what it is you're testing, but can omit context if you don't think it's necessary.

A popular assertion library for JavaScript is called [Chai](http://chaijs.com/). This is what we're going to use. It's already been added to the test template.

Modify your `test.js` to the following:

```js
it("can assert true", function(){
  assert(true, 'TRUE IS FALSE! UP IS DOWN! DAY IS NIGHT!');
});
```

Like RSpec and minitest, I can have an optional message as part of my assertion which will be displayed if the assertion fails. If `true` is somehow false, something has seriously gone wrong.

When you refresh your test page, you should see the test pass.

Let's try out a few more of Chai's assertions.

```js
describe("Chai Assertions Sandbox", function(){
  it("can assert true", function(){
    assert(true);
  });

  it("can assert 1 is 1", function(){
    assert.equal(1, 1);
  });

  it("can assert 2 is not 3", function(){
    assert.notEqual(2, 3);  
  });

  it("can assert that something is a given data type", function(){
    assert.isNumber(42);
    assert.isObject({answer: 42});
    assert.isArray([1,2,3,4]);
    var thingIHaventDefined;
    assert.isUndefined(thingIHaventDefined);
  });
});
```

#### `equal`, `strictEqual` and `deepEqual`

Equality in JavaScript is funky. `1 == true` but not `1 === true`. `'3' == 3` but not `'3' === 3`. `assert.equal` will compare using double equals (`==`), and `assert.strictEqual` will compare using triple equals(`===`).

`deepEqual` is used for arrays and objects. In the deep underpinnings of JavaScript, each time you define an array, it's a different array. So `[1,2,3,4] == [1,2,3,4]` will always return false. `deepEqual` will compare each value in an array, or each key/value pair in an object. Let's add the following to our sandbox:

```js
    it("can compare two arrays that contain the same values", function(){
      var actualArray = [1,2,3,4];
      assert.deepEqual(actualArray, [1,2,3,4]);
    })

    it("can compare two objects that contain the same key/value pairs", function(){
      var actualObject = {name: "Nate", module: 4};
      assert.deepEqual(actualObject, {name: "Nate", module: 4});
    })
```

Unit testing
--------------------

For our purposes, unit testing is verifying that a function will return a certain value given certain inputs. Unit tests are great for functions that perform logic for you, but don't involve user interaction directly. Here's an example:

```js
describe("We can square a number", function(){
  it("returns 4 when I pass it 2", function(){
    assert.equal(square(2), 4);
  });
});
```

We're testing that when I pass the function `square()` a value of `2`, then I get a return value of `4`. Since we're using TDD, we don't yet have an implementation of `square()`, so we need to build one. We could define `square()` in the test file itself, but that doesn't make it very portable.

Let's create `lib/square.js`. Then we can reference that file in our test HTML by adding this to our `<head>`:

```HTML
<script src="../lib/square.js"></script>
```

Now, make the test pass.

### A few more useful mocha features

-   Just like RSpec, and even Minitest, Mocha supports `before()`, `beforeEach()`, `after()` and `afterEach()` inside of a `describe()` or `context()`.
-   If you'd like to skip a test, you can mark it as pending by changing `it()` to `xit()`.

Let's integrate
-------------

So, what do we need to do to successfully automate our integration tests?

### Break it down

- Create our setup conditions
- Access the page that has our features
- Interact with the page that has our features
- Determine the current state of things
- Be able to compare the actual state of things with our expected state of things
- Clean up any changes that were made

Here's the tools we're going to use for each of these:

- **mocha (before and beforeEach)**: Create our setup conditions
- **an iframe**: Access the page that has our features
- **jQuery (.click, .trigger)**: Interact with the page that has our features
- **jQuery (.html, .text, .attr, etc)**: Determine the current state of things
- **chai (assert)**: Be able to compare the actual state of things with our expected state of things
- **mocha (after and afterEach)**: Clean up any changes that were made

### Build it up

```js
describe('#create-form', function() {
  var $;

  before(function(){
    $ = document.getElementById("foods-frame").contentWindow.$;
  })

  beforeEach(function() {
    //Clear out all the things
    $('#food-list tbody').html('');
    $('#create-form input').val('');
    $('.validation-error').html('');
  });

  context('validations', function() {

    it('will tell me if I fail to enter a name', function() {
      $('#calories-field input').val('35');
      $('#add-food').click();
      var nameValidationContent = $("#name-field .validation-error").text();
      assert.equal(nameValidationContent, "Please Enter a Name");
    });

    it('will tell me if I fail to enter calories', function() {
      $('#name-field input').val('Banana');
      $('#add-food').click();
      var caloriesValidationContent = $("#calories-field .validation-error").text();
      assert.equal(caloriesValidationContent, "Please Enter Calories");
    });

    it('will be nice to me if I do everything correctly', function() {
      $('#name-field input').val('Banana');
      $('#calories-field input').val('35');
      $('#add-food').click();

      var nameValidationContent = $("#name-field .validation-error").text();
      assert.equal(nameValidationContent, "");

      var caloriesValidationContent = $("#calories-field .validation-error").text();
      assert.equal(caloriesValidationContent, "");
    });

  });
});
```

How are we using each of the parts in the breakdown in the test above?

Now try writing another test for this user story:


### Some hints as you continue

- Using jQuery, there's nothing as nice as `.fill`. We have to trigger each keydown and pass in the key you're pressing. I recommend writing a function that will take a string, and fire a keydown for each character in the string. I think this [StackOverflow answer](http://stackoverflow.com/a/832121/4075893) spells out the details pretty well
- The code in the `before` function gives you access to jQuery on the page you're testing, but you may need to do similar stuff to get access to other things, like `localStorage`.
