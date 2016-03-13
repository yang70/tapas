# Message Encryptor - A Single Page Javascript Encryption Application

By [Matthew Yang](http://www.matthewgyang.com).

## Description
This application was created as a class assignment and was modified from the base repo located [here](https://github.com/brookr/encryptor).  We were to modify this code into a working application and set up a testing suite.

## My Approach
My first step was to examine the code and see what the intent was.  I made a decision to use 'vanilla' javascript rather than jQuery for more pure practice.  I then filled in the `encryptor` object and it's methods with appropriate code.  In order to make the application adaptable to different domains, I decided to grab the current url via `window.location.href` rather than hard code it.  I then concatenated a "?" followed by the encrypted message cypher.

The dynamic page content is keyed by a javascript function that runs on page load that checks if there is a "?" in the current url via `window.location.href.includes('?')`.  This is by no means a perfect system, and breaks down if someone types in a question mark into the url manually.  However it works most of the time and is an acceptable solution in my opinion.

## Testing
Testing was the most difficult aspect of this assignment.  I set up testing with [NPM](https://www.npmjs.com), [Mocha](https://mochajs.org) and [Chai](http://chaijs.com) (`node_modules` has been git-ignored so they won't be shown in this repository).  The problem was how to test this (without having to use something like Selenium Webdriver).  I solved this by extracting the actual encryption code from the main js file in order to isolate it from all of the browser dependant javascript.  That way I can test that the encrypt/decrypt function is working.

The other obstacle was the [CryptoJS](http://crypto-js.googlecode.com/svn/tags/3.1.2/build/rollups/aes.js) encryption source.  For some reason I could not get it to load in the test file and ended up just including the entire code base in my `encryptor.js` file.

## Credit
* Base code source [https://github.com/brookr/encryptor](https://github.com/brookr/encryptor/blob/master/index.html)
* CryptoJS [https://code.google.com/p/crypto-js/](https://code.google.com/p/crypto-js/)
