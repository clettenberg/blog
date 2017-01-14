---
layout: post
title: Test `run` block in AngularJS
date: 2017-01-13
author: Chase Clettenberg
author_login: clettenberg
author_email: clettenberg@gmail.com
author_url: http://cclettenberg.com
categories: angular-js
tags: angular-js tdd
---

Here is an example of setting up event listeners in a module's `run` block:

```js
angular.module('myApp', [])
.run(function($window, $log) {
  $window.addEventListener('offline', () => {
    $log.info('You are offline');
  });

  $window.addEventListener('online', () => {
    $log.info('You are online');
  });
});
```

First you have to `$provide` your mock to the module and then `inject()` without any arguments
in order to make assertions on that mock.

```js
describe('run', function () {
  describe('when the app is initialized', function () {
    var mockWindow;

    beforeEach(function() {
      mockWindow = {
        addEventListener: sinon.spy(),
          dispatchEvent: sinon.spy()
        };

      module('darts', function($provide) {
        $provide.value('$window', mockWindow);
      });
    });

    it('should add the appropriate network status event listeners', function() {
      inject(); // This is what makes it work
      expect(mockWindow.addEventListener).to.have.been.calledWith('offline');
      expect(mockWindow.addEventListener).to.have.been.calledWith('online');
      );
  });
});
```
