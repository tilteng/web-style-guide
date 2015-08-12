# Flux Patterns

Flux is an application architecture where global data is put into stores, the views render data from the stores.  On user action, the views invoke action creators, which update the stores, and the views re-render based on the new state of the stores.

Recommended Reading:
* https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0 
* https://medium.com/@dan_abramov/the-evolution-of-flux-frameworks-6c16ad26bb31
* http://gaearon.github.io/redux/docs/recipes/ReducingBoilerplate.html

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

## Stores

A store is a global representation of data that "backs" a view.

Stores are global data.  Global data doesn't have to be a bad thing if controlled and updated in a predictable way.  Stores extract the "backing data" of a view into a global place which allows for easy access and a predictable way of responding.

 With this separation of data, it's really easy to handle "live update" cases.  The page can connect to a websocket and wait for push notifications in the background of updated campaign data.  On a socket notification, the store emits change and the view is rerendered with the new data.  (XHR polling can also be used to accomplish the same thing.)

Stores similarly make "faking data" before a webservice call has completed really straightforward.  When the user clicks 'send' to create a comment, a fake comment is created in the store, and the store signals a change has occurred.  The view rerenders, showing the new comment.  After the comment has been successfully persisted, the store is updated to replace the fake comment's data with the real comments data, and the store emits change again, updating the new comment with its real backing data.  This level of responsiveness is expected by most customers in their interaction with the social features on a website.

### Best Practices

The best implementation of a store:
* corresponds to a `GET` endpoint from some upstream service (API)
* stores data is completely parameterized by the `GET` arguments to that endpoint

A store backing the Tilt page - upstream API route `/tilts/:id`, would contain data inside of it parameterized by Tilt `id`.  This allows for storing data for multiple tilts in memory at once, which allows seamless transitions between different tilts without a page refresh.  As an example, we could preload data for a tilt when a user mouses over a notification in their notification center - on click, the page could be loaded automatically from memory, avoiding a loading screen.  

This can lead to somewhat silly conclusions: "`/users/:id/notification` should be parameterized by user".  At first thought this is a bad conclusion, because only one user is ever logged in at once.  However, a store that is parameterized by user id would allow quickly switching between "active accounts", or displaying data from multiple accounts on the screen at once.  Building a store like this for present product requirements may be overdesign (and so not worth it) but it is good to think through these edge cases and understand the tradeoff that is being made ;)

### Try To Avoid

Try to avoid the store pattern for:
* data that better represents view state, e.g. errors temporarily displayed, submission in progress, or successfully finished
* data that will never change (this is better owned and passed down as props from some parent component)

Using stores to hold UI state is a bit of an antipattern.  Sometimes this is useful but what you're really getting out of it is the global access to the data.  This data is better passed down as props from a parent component rather than having a leaf component access the store.  If UI state needs as the result of a user action (e.g. a Flux action creator), consider passing a callback down from a parent component instead.

For example, rather that using a UI store to keep track of whether state is submitting or not, pass a callback to a child component that is used to invoke the submit which updates the state of the child component.

```javascript
const Parent = React.createClass({

	_submitForm(data) {
		this.setState({ submitting: true });
		Actions.doVeryLongAction(data).then(() => {
			this.setState({submitting: false})
		});
	},
	
	render() {
		return <div>
			<Child 
				submitting={this.props.submitting}
				submitForm={this._submitForm} />
		</div>;
	}

});
```

This factoring is much easier both to unit test (because the Child receives everything as props) and to test in the browser using the React dev tools (because you can test submitting state on the child component without actually invoking an external action).

## Dispatcher

We use Yahoo's [dispatchr](https://github.com/yahoo/dispatchr) library to do server-side rendering.  Overuse of the dispatcher should be avoided because it signifies that your components may be reaching too much into global state.

* Pass name of the store to retrieve it (```dispatcher.getStore('SomeStore')```), instead of passing the interface
