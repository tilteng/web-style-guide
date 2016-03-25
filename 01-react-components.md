# React Components

## Basics
* Use displayName
* Use propTypes
* Avoid the use of state, prefer props
* Prefix any function not part of React life cycle methods with an underscore ```_``` character


## Props

```javascript
const { currentUser } = this.props;

currentUser
currenUser // Lint picks up this typo
currentUser
```
vs
```javascript
this.props.currentUser
this.props.currenUser // Lint does not pick up the typo
this.props.currentUser
```
