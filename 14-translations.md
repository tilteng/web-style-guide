# Translations


## Usage
#### Singular
```javascript
<LocalizedMessage message={_('Hello')} />
```

#### Plural
```javascript
const friendCount = ...;

<LocalizedMessage
  numberOfFriends={<LocalizedNumber number={friendCount} />}
  message={_n(
    'Facebook ({numberOfFriends} friend)',
    'Facebook ({numberOfFriends} friends)',
    friendCount
  )} />
```

#### Singular with Context
```javascript
<LocalizedMessage message={_c('flag', 'flag comment for delete')} />
```

#### Plural with Context
```javascript
const count = ...;
<LocalizedMessage
  message={_nc('Like', 'Likes', count, 'comment likes')} />
```

## Translator Functions
```javascript
import { _, _n, _c, _nc } from 'web/root/translators';
```

These mimic standard gettext functions, and all return a function. These returned functions must be invoked with a locale which essentially evaluate a translation. This is done for you in `LocalizedMessage` which has a locale in the context, otherwise you must do it yourself explicitly:
```javascript
const locale = ...;
<img alt={_('Hello')(locale)} />
```

## Extraction
Using one of the functions above will mark a string for extraction into a standard gettext PO file. To extract strings, run `gulp extract`. You must pass strings directly into the functions for them to be captured.

## Messages
A generated PO file is not used directly by server or client, and must first be converted into JSON. To generate messages for the client/server, run `gulp messages`.
