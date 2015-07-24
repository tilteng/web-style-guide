# Unit Tests

## Test Names
* Prefix test names appropriately:
  * ```#``` means that it's describing an instance method
  * ```.``` means that it's a class method
  * nothing means it's describing something else
  * With the above rules we can expect output as follows:
  ```
  - HtmlUtils:
      - #hasClass:
          should work if the browser supports classList PASSED
          should work if the browser doesn't support classList PASSED
  ```

## Assertions
* Use ```expect``` assertion style with Chai instead of ```should``` or ```assert```
* Use the most strict form of assertion (```expect``` chain) you can. For example:
  * If testing ```Form#isValid()```, use ```to.be.true``` instead of ```to.be.ok```
  * If testing ```React.findDOMNode(component)```, use ```to.exist``` instead of ```to.be.ok```
* Refer to a cheatsheet like this [one](http://ricostacruz.com/cheatsheets/chai.html)

## Promises
```javascript
it('dispatches a userResetInfo action after success', function(done) {
   server.respondWith(...);
   
   SessionActions.refreshSessionUser(dispatcher).then(function() {
       expect(dispatcher.dispatch).to.have.been.calledWith('userResetInfo', response);
       done();
   });
   
   server.respond();
}).catch(done);
```
* When testing promises directly:
  * Add a ```done``` callback to let framework know it should wait for completion
  * Invoke the callback to signal this completion inside the Promise callbacks
  * You must have a final ```.catch(done)``` (or some other form) to hand over an assertion error (if one occurs) to test framework
