# selenium-cucumber-js

[![Shippable branch](https://img.shields.io/shippable/5818b23bbd56670e00037040/master.svg)](https://app.shippable.com/projects/5818b23bbd56670e00037040) [![npm](https://img.shields.io/npm/dt/selenium-cucumber-js.svg)](https://www.npmjs.com/package/selenium-cucumber-js) [![Linked In](https://img.shields.io/badge/Linked-In-blue.svg)](https://www.linkedin.com/in/john-i-doherty) [![Twitter Follow](https://img.shields.io/twitter/follow/CambridgeMVP.svg?style=social&label=Twitter&style=plastic)](https://twitter.com/CambridgeMVP)

JavaScript browser automation framework using official [selenium-webdriver](http://seleniumhq.github.io/selenium/docs/api/javascript/ "view webdriver js documentation") and [cucumber-js](https://github.com/cucumber/cucumber-js "view cucumber js documentation").

## Installation

```bash
npm install selenium-cucumber-js --save-dev
```

## Usage

```bash
node ./node_modules/selenium-cucumber-js/index.js -s ./step-definitions
```

### Options

```bash
-h, --help                   output usage information
-V, --version                output the version number
-s, --steps <path>           path to step definitions. defaults to ./step-definitions
-p, --pageObjects <path>     path to page objects. defaults to ./page-objects
-o, --sharedObjects [paths]  path to shared objects (repeatable). defaults to ./shared-objects
-b, --browser <path>         name of browser to use. defaults to chrome
-r, --reports <path>         output path to save reports. defaults to ./reports
-d, --disableLaunchReport    disable the auto opening the browser with test report
-j, --junit <path>           output path to save junit-report.xml defaults to ./reports
-t, --tags <tagName>         name of tag to run
-f, --featureFile <path>     a specific feature file to run
-x, --timeOut <n>            steps definition timeout in milliseconds. defaults to 10 seconds
```

By default tests are run using Google Chrome, to run tests using another browser supply the name of that browser along with the `-b` switch. Available options are:

| Browser | Example |
| :--- | :--- |
| Chrome | `-b chrome` |
| Firefox | `-b firefox` |
| Phantom JS | `-b phantomjs` |

### Feature files

A feature file is a [Business Readable, Domain Specific Language](http://martinfowler.com/bliki/BusinessReadableDSL.html) file that lets you describe software’s behaviour without detailing how that behaviour is implemented. Feature files are written using the [Gherkin syntax](https://github.com/cucumber/cucumber/wiki/Gherkin) and must live in a folder named **features** within the root of your project.

```gherkin
# ./features/google-search.feature

Feature: Searching for vote cards app
  As an internet user
  In order to find out more about the itunes vote cards app
  I want to be able to search for information about the itunes vote cards app

  Scenario: Google search for vote cards app
    When I search Google for "itunes vote cards app"
    Then I should see some results
```

The browser automatically closes after each scenario to ensure the next scenario uses a fresh browser environment.

### Step definitions

Step definitions act as the glue between features files and the actual system under test.

_To avoid confusion **always** return a JavaScript promise your step definition in order to let cucumber know when your task has completed._

```javascript
// ./step-definitions/google-search-steps.js

module.exports = function () {

    this.Then(/^I should see some results$/, function () {

        // driver wair returns a promise so return that
        return driver.wait(until.elementsLocated(by.css('div.g')), 10000).then(function(){

            // return the promise of an element to the following then.
            return driver.findElements(by.css('div.g'));
        })
        .then(function (elements) {

            // verify this element has children
            expect(elements.length).to.not.equal(0);
        });
    });
};
```

The following variables are available within the ```Given()```, ```When()``` and ```Then()``` functions:

| Variable | Description |
| :--- | :--- |
| `driver`    | an instance of [selenium web driver](http://seleniumhq.github.io/selenium/docs/api/javascript/module/selenium-webdriver/lib/webdriver_exports_WebDriver.html) (_the browser_) |
| `selenium`  | the raw [selenium-webdriver](http://seleniumhq.github.io/selenium/docs/api/javascript/module/selenium-webdriver/) module, providing access to static properties/methods |
| `page`      | collection of **page** objects loaded from disk and keyed by filename |
| `shared`    | collection of **shared** objects loaded from disk and keyed by filename |
| `helpers`   | a collection of [helper methods](runtime/helpers.js) _things selenium does not provide but really should!_ |
| `by`        | the selenium [By](http://seleniumhq.github.io/selenium/docs/api/javascript/module/selenium-webdriver/index_exports_By.html) class used to locate elements on the page |
| `until`     | the selenium [until](http://seleniumhq.github.io/selenium/docs/api/javascript/module/selenium-webdriver/lib/until.html) class used to wait for elements/events |
| `expect`    | instance of [chai expect](http://chaijs.com/api/bdd/) to ```expect('something').to.equal('something')``` |
| `assert`    | instance of [chai assert](http://chaijs.com/api/assert/) to ```assert.isOk('everything', 'everything is ok')``` |
| `trace`     | handy trace method to log console output with increased visibility |



### Page objects

Page objects are accessible via a global ```page``` object and are automatically loaded from ```./page-objects``` _(or the path specified using the ```-p``` switch)_. Page objects are exposed via a camel-cased version of their filename, for example ```./page-objects/google-search.js``` becomes ```page.googleSearch```.

Page objects also have access to the same runtime variables available to step definitions.

An example page object:

```javascript
// ./page-objects/gogole-search.js

module.exports = {

    url: 'http://www.google.co.uk',

    elements: {
        searchInput: by.name('q'),
        searchResultLink: by.css('div.g > h3 > a')
    },

    /**
     * enters a search term into Google's search box and presses enter
     * @param {string} searchQuery
     * @returns {Promise} a promise to enter the search values
     */
    preformSearch: function (searchQuery) {

        var selector = page.googleSearch.elements.searchInput;

        // return a promise so the calling function knows the task has completed
        return driver.findElement(selector).sendKeys(searchQuery, selenium.Key.ENTER);
    }
};
```

And its usage within a step definition:

```js
// ./step-definitions/google-search-steps.js
this.When(/^I search Google for "([^"]*)"$/, function (searchQuery) {

    return helpers.loadPage('http://www.google.com').then(function() {

        // use a method on the page object which also returns a promise
        return page.googleSearch.preformSearch(searchQuery);
    })
});
```

### Shared objects

Shared objects allow you to share anything from test data to helper methods throughout your project via a global ```shared``` object. Shared objects are automatically loaded from ```./shared-objects``` _(or the path specified using the ```-o``` switch)_ and made available via a camel-cased version of their filename, for example ```./shared-objects/test-data.js``` becomes ```shared.testData```.

Shared objects also have access to the same runtime variables available to step definitions.

An example shared object:

```javascript
// ./shared-objects/test-data.js

module.exports = {
    username: "import-test-user",
    password: "import-test-pa**word"
}
```

And its usage within a step definition:

```js
module.exports = function () {

    this.Given(/^I am logged in"$/, function () {

        driver.findElement(by.name('usn')).sendKeys(shared.testData.username);
        driver.findElement(by.name('pass')).sendKeys(shared.testData.password);
    });
};
```

### Reports

HTML and JSON reports are automatically generated and stored in the default `./reports` folder. This location can be changed by providing a new path using the `-r` command line switch:

![Cucumber HTML report](img/cucumber-html-report.png)

### Event handlers

You can register event handlers for the following events within the cucumber life cycle.

| Event          | Example                                                     |
|----------------|-------------------------------------------------------------|
| BeforeFeature  | ```this.BeforeFeatures(function(feature, callback) {}) ```  |
| BeforeScenario | ```this.BeforeScenario(function(scenario, callback) {});``` |
| AfterScenario  | ```this.AfterScenario(function(scenario, callback) {});```  |
| AfterFeature   | ```this.AfterFeature(function(feature, callback) {});```    |

## How to debug

Most selenium methods return a [JavaScript Promise](https://spring.io/understanding/javascript-promises "view JavaScript promise introduction") that is resolved when the method completes. The easiest way to step in with a debugger is to add a ```.then``` method to a selenium function and place a ```debugger``` statement within it, for example:

```js
module.exports = function () {

    this.When(/^I search Google for "([^"]*)"$/, function (searchQuery, done) {

        driver.findElement(by.name('q')).then(function(input) {
            expect(input).to.exist;
            debugger; // <<- your IDE should step in at this point, with the browser open
            return input;
        })
        .then(function(input){
            input.sendKeys(searchQuery);
            input.sendKeys(selenium.Key.ENTER);

            done(); // <<- let cucumber know you're done
        });
    });
};
```

## Default directory structure

You can use the framework without any command line arguments if your application uses the following folder structure:

```
.
├── features
│   └── google-search.feature
├── step-definitions
│   └── google-search-steps.js
├── page-objects
│   └── google-search.js
└── shared-objects
│   ├── test-data.js
│   └── stuff.json
└── reports
    ├── cucumber-report.json
    └── cucumber-report.html
```

## Demo

This project includes an example to help you get started. You can run the example using the following command:

```bash
node ./node_modules/selenium-cucumber-js/index.js
```

## Bugs

Please raise bugs via the [selenium-cucumber-js issue tracker](https://github.com/john-doherty/selenium-cucumber-js/issues) and, if possible, please provide enough information to allow the bug to be reproduced.

## Contributing

Everyone is very welcome to contribute to this project. You can contribute just by submitting bugs or suggesting improvements by [opening an issue on GitHub](https://github.com/john-doherty/selenium-cucumber-js/issues).

## Troubleshooting

### IntelliJ Cucumber Plugin

IntelliJ based IDE's have a plugin that allows the tester to control click on a `Given`, `When`, `Then` statement within a Cucumber feature file and have the user taken to the associated step definition. This plugin relies on your project having the following folder structure:

```
.
└── features
   │   google-search.feature
   └── step_definitions
   │   └── google-search-steps.js
   └── page_objects
   │   └── google-search.js
   └── shared_objects
   │   ├── test-data.js
   │   └── stuff.json
   └── reports
       ├── cucumber-report.json
       └── cucumber-report.html
```

This can be achieved by restructuring your project to match the layout above _(notice the underscores)_, and running your tests with the following switches:

```bash
node ./node_modules/selenium-cucumber-js/index.js -s ./features/step_definitions -p ./features/page_objects -o ./features/shared_objects -r ./features/reports
```

## License

Licensed under [ISC License](LICENSE) &copy; [John Doherty](http://www.johndoherty.info)