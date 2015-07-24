# Flux Patterns

## Actions
```javascript
const request = require('superagent');
const RequestUtils = require('site/utils/RequestUtils');
const UserConstants = require('site/constants/UserConstants');

const SessionActions = {
    refreshSessionUser: function(dispatcher) {
        const userStore = dispatcher.getStore('UserStore');

        if (userStore.isLoggedIn()) {
            return RequestUtils
                .get('/ajax/session-user-refresh')
                .then(function(response) {
                    dispatcher.dispatch(UserConstants.actions.USER_RESET_INFO, response.body);
                });
        } else {
            return Promise.reject(new Error('User is not logged in'));
        }
    }
};

module.exports = SessionActions;

```
### Basics
* Pass the dispatcher as an argument
* Define action names in a ```(x)Constants.js``` files

### Promises
* Use Promises instead of callbacks. They're easy to chain and error handling is clean
  * Use our ```RequestUtils``` for utility methods to create requests in Promises

## Dispatcher
* Pass name of the store to retrieve it (```dispatcher.getStore('SomeStore')```), instead of passing the interface

## Stores
