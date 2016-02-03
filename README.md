# awesome guide to angular protractor testing

some notes and lessons learned writing protractor tests and getting them to run on ci.

### style and code cleanliness

1. use coffeescript
2. assign variables shared across `it` blocks to `this`

  ```coffee
  # good
  beforeEach ->
    @foo = 1

  # bad
  foo = null
  beforeEach ->
    foo = 1
  ```
3. use DSLs and PageObjects copiously

  ```coffee
    # good
    class MyPage
      getButton: -> $('#my-button')
      getForm: -> $('#my-form')
      enterTextInForm: (text) ->
        @getButton().click()
        @getForm().$$('input').sendKeys text

    beforeEach -> @myPage = new MyPage

    it 'should foo', ->
      @myPage.enterTextInForm('foo')
      expect(foo).toBe('foo')

    it 'should bar', ->
      @myPage.enterTextInForm('bar')
      expect(foo).toBe('bar')

    # bad
    it 'should foo', ->
      $('#my-button').click()
      $$('#my-form input').get(0).sendKeys 'foo'
      expect(foo).toBe('foo')

    it 'should bar', ->
      $('#my-button').click()
      $$('#my-form input').get(0).sendKeys 'bar'
      expect(foo).toBe('bar')
  ```

### notes

- sometimes protractor will have trouble synchronizing. there are a few ways to wait for protractor to synchronize:
  - `browser.wait(-> $$('.foo').isDisplayed(), 1000)`
  - `browser.waitForAngular()`
  - `browser.driver.sleep(1000)`
- you can reach into appland and execute javascript code directly:
  - `browser.executeScript()`
  - `browser.executeScriptAsync((callback) -> ...)`
  - `browser.addMockModule('myModule', angular.module('myModule')...)`

### debugging tests

- use `fdescribe`, just like unit tests
- log stuff out to the terminal (`console.log`), just like unit tests
- pause the test and inspect the state of the app (`browser.enterRepl()`)
- query for elements' internal state (`$(mySelector).evaluate('someScopeProperty')`)
- see if a timeout fixes it (`browser.driver.sleep(5000)`)
- read out the browser's logs by first setting a breakpoint (`browser.enterRepl()`), then:

   ```js
   browser.manage().logs().get('browser').then(browserLog =>
     console.log(require('util').inspect(browserLog))
   )
   ```

### setting up protractor on ci environments (eg. ubuntu)

- see this guide: https://www.exratione.com/2013/12/angularjs-headless-end-to-end-testing-with-protractor-and-selenium/