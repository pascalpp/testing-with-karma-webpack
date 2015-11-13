<!-- .slide: data-state="intro" -->
# Writing and Running Tests

Notes:

!!

## Some Caveats

- ### Our tests are a work-in-progress
- ### I’m not an expert
- ### We’re all learning as we go

!!

## Some Vocabulary

### Karma

Our test runner

### Webpack

Our module bundler / compiler

### Mocha

Our testing framework

!

## More Vocabulary

### Unit testing

Testing the individual parts of a feature, separately from each other.
This is the focus of this talk.

### Integration testing

Testing collections of units working together.<br>Testing whole UI flows, including multiple screens.

!!

<!-- .slide: data-state="filesystem" -->
## The Lay of the Land

```
main/
  client/
    src/
      index.js
    test/
      tests.js
      tests/
    package.json
    bower.json
    webpack.config.js
    karma.conf.js
    config/
```

!!

## Setting up

```
cd client
npm install
bower install
npm install -g karma-cli webpack
```

!!

<!-- .slide: data-state="testexample" -->
## Running Tests with Karma

<img src="./images/testoutputexample.png">

`karma start`

- Loads and compiles our tests<br />(using webpack under the hood)
- Starts a webserver on port 9876
- Points one or more browsers at that server
- Any UI created by your tests appears in the browser (except PhantomJS)
- Test results appear in the terminal where karma is running


!

<!-- .slide: data-state="continuous" -->
## Running Tests Continuously

- By default, karma keeps running, waiting for changes
- Save changes, tests rerun automatically
- Incremental recompiles, much faster
- Point any browser at &lt;yourip&gt;:9876
- <img src="./images/phone.png" style="float:right;width:400px;height:auto;margin-top:15px;">karma will 'capture' it for testing — even your phone!

!

## Running Tests Once

`karma start --single-run`

- Compiles, runs the tests, and quits

!

## Running Tests In Different Browsers

`KARMA_BROWSERS=Chrome,Safari karma start`

- our config parses the KARMA_BROWSERS environment variable
- Browsers can be: Chrome, Safari, Firefox, or PhantomJS
- Karma just uses PhantomJS by default
- Nicholas is working on BrowserStack integration, stay tuned

!

## Running Tests For Production

`NODE_ENV=production karma start`
- tells webpack to compile for production<br>(minification, dead-code removal, etc)
- tells karma to run once (same as `--single-run`)
- uses 'dots' reporter instead of checklist
- slower, not recommended for development

!!

## Writing tests

!!

# NEEDLE SCRATCH!

!!

## Writing testable code

A few thoughts…

!

### Programming is not writing code

!

### Programming is problem-solving

!

### Every feature is a problem to be solved

!

### Complex problems are solved<br>by breaking them<br>into smaller,<br>simpler<br>problems

!

### …code is just an artifact of the solution

!

### The more you break the problem down,<br>the simpler each code artifact is

!!

## Figuring out the parts

!

## Key questions to ask yourself:

- What are the parts of this feature?
- What are the responsibilities of each part?
- Do any parts have too many responsibilities?
- How can I test each part independently from the others?

!

## A feature is a hierarchy of parts

- A large part which relies on smaller parts
- Those smaller parts might rely on still smaller parts

<img src="./images/parts.png" style="height:300px;width:auto;">

!

## Breaking apart the parts

- Make each part an independent (requirable) module
- Give each part of a feature one 'job', ideally
- Accomplishing that job might involve telling other smaller parts to do their job
- Ideally, no part acts of its own volition—<br>some larger part tells it when to do its job

### Smaller parts make writing tests easier

!

## Gotchas and Alternatives

- Don’t make assumptions about window.AboutMe or its properties
    - viewer
    - viewport
    - browser
- Don’t use those objects to _define_ your module
- Instead, listen for changes in those objects _inside_ your module

- Avoid using AboutMe.on, when, trigger, request, execute
- Use radio channels instead - we have work to do here

!!

## Writing a test suite in mocha

### A suite is a collection of tests

- `describe` an object
    - `it` should
    - `it` should not
- Use nested describe blocks for an object’s properties and methods

!

## Setting up and tearing down

### A test suite should be able to:
- run independently of other suites
- run in conjunction with other suites

!

## before and after

- `before` your tests run, set up requirements
    - instantiate assumed globals (ahem)
    - mock objects
    - require modules needed in your test
- `after` your tests run, tear down anything you set up
    - delete window globals
    - remove dom elements

!

## beforeEach and afterEach

- set up and tear down for each test in your suite
- use these within `describe` blocks to simplify your `it` statements

!

<!-- .slide: data-state="examplesuite" -->
## An example test suite

```
describe('module/foo/foo_view', function() {

    // define vars used throughout your suite
    var FooView
    var foo_view
    var node
    var region

    // don't perform any logic directly in your describes
    // use before and after
    before(function() {
        require('mocks/globals') // sets up a mock globals json glob
        FooView = require('module/foo/foo_view')
        node = $('<div />').appendTo('body')
        region = new Marionette.Region({ el: node })
    })
    after(function() {
        region.destroy()
        node.remove()
    })

    // create a fresh foo_view for each test
    beforeEach(function() {
        foo_view = new FooView()
        region.show(foo_view)
    })

    // describe each part of your view or object

    // write pending tests first
    // using it('should do something') with no callback
    // implement the test callback later

    describe('#ui', function() {
        describe('#headline', function() {
            it('should exist')
        })
        describe('#button', function() {
            it('should exist')
        })
    })

    describe('#someMethod', function() {
        it('should do something')
        it('should not do something')
    })

})
```

!!

## Implementing test logic with chai

`expect` and `should`

```
var chai = require('chai')
var expect = chai.expect
chai.should()
```

!

## Expect and Should

```
describe('#ui', function() {
    describe('#headline', function() {
        it('should exist', function() {
            expect(foo_view.ui.headline).to.exist
        })
    })
    describe('#button', function() {
        it('should exist', function() {
            foo_view.ui.button.should.exist
        })
    })
})
```

!!

<!-- .slide: data-state="sinon" -->
## Stubbing and spying with sinon

- Replace a function with sinon.stub

    `sinon.stub(SomeObject, 'someMethod')`

- Attach a rider with sinon.spy, allows the original function to run

    `sinon.spy(SomeObject, 'someMethod')`

- Don't forget to restore when you’re done

    `SomeObject.someMethod.restore()`


!

<!-- .slide: data-state="sinon" -->
## Stubbing and spying with sinon

```
var sinon = require('sinon');

describe('MyObject', function() {
    describe('#myMethod', function() {
        beforeEach(function() {
            sinon.stub(OtherObject, 'otherMethod')
        })
        afterEach(function() {
            Object.otherMethod.restore()
        })
        it('should call OtherObject.otherMethod', function() {
            MyObject.myMethod()
            OtherObject.otherMethod.calledOnce.should.be.true
        })
    })
})
```

!!

## Mocking Ajax calls

Haven’t used this myself yet, but:

```
require('jquery.ajaxmock')
$.ajaxMock.register('/n/3/internal/url_exists', {
    responseText: '{ exists: true }',
    statusCode: 200,
    status: 'OK',
    type: 'POST', // optional, default: 'GET'
    delay: 1000 // optional
})
```

- Test how your object handles different responses
- Test slow responses!

!!

## More to explore

### rewire-webpack

- https://github.com/jhnns/rewire-webpack
- Dependency injection for webpack bundles

!!

## Adding your test suite

- Write your test in a location similar to the src file being tested
    ```
/client/src/module/mymodule/mymodule.js
/client/test/tests/module/mymodule.test.js
```

- Require your test in `client/test/tests.js`

    `require('./tests/module/mymodule.test.js')`

- Comment out other tests to see if your test runs independently

- Don’t commit with other tests disabled!

!!

## Other things at your disposal

- The main tests file requires `module/app/bootstrap`
- So you can use $, \_, Backbone, and Marionette in your tests
- window.log is also provided, helps when debugging in PhantomJS
- `log('object.foo', object.foo)`<br>appears inline in karma results output
- don’t commit log statements in tests

!!

## Dealing with reality

- Much of our current codebase is built on assumptions
- Lots of interdependencies
- Can be difficult to untangle
- Situation is improving; you can help

!

## Untangling the yarns

- In your test, try to require just the module you’re testing
- See what missing dependencies and assumptions the module complains about
- Fix the breakage by decoupling code
- Or, satisfy the requirement in your test

!

<!-- .slide: data-state="untangle" -->
## Decoupling code with require.ensure

### Try turning this:

```
require('module/compliment_modal/compliment_modal');

var MyView = Marionette.LayoutView.extend({

    someMethod: function() {
        AboutMe.trigger('user:compliment:ui');
    }

});
```

!

<!-- .slide: data-state="untangle" -->
### …into this:

```
var MyView = Marionette.LayoutView.extend({

    someMethod: function() {
        require.ensure([], function() {
            require('module/compliment_modal/compliment_modal');
            AboutMe.trigger('user:compliment:ui');
        })
    }

});
```

Now we can test our view without loading compliment_modal<br>and all of its dependencies.

!

## Satisfying common requirements in our codebase

- globals (required by lib/log, which is required all over the place)

    `require('mocks/globals')`

- other dom json (required by lib/analytics, for example)

    ```
var insertDomJson = require('mocks/insert_dom_json')
insertDomJson({}, 'json analytics')
```

!

## F*&k it, start the app

Sometimes you just can’t test a unit outside the app context. Too many dependencies to untangle, maybe now isn’t the time to untangle them.

```
before(function() {
    var MockAboutMe = require('mocks/aboutme')
    var app = MockAboutMe.setup()
    app.start()
})

after(function() {
    MockAboutMe.teardown()
})
```

!!


## Getting help

- Misery loves company
- Join #fed-testing in Slack
- Use Screenhero to pair with coworkers and write tests together

!!

<!-- .slide: data-state="end" -->
# Happy Testing!
