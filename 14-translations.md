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

These mimic standard gettext functions, and all return a function. These returned functions must be invoked with a locale. This is done for you in `LocalizedMessage` which has a locale in the context, otherwise you must do it yourself explicitly:
```javascript
const locale = ...;
<img alt={_('Hello')(locale)} />
```

## Translator Components
Both components return a `span` element since React does not allow components to return a raw string.

```javascript
<LocalizedMessage timeAgo={<LocalizedTimeAgo date={...} />} message={_('You ate {timeAgo}!')} />
```
This component which can accept placeholder values of any type, including React nodes.

```javascript
<LocalizedHTMLMessage name={'<span class="highlight">Bob</span>'} message={_('Hello {name}!')} />
``` 
This component sets HTML directly using ```dangerouslySetInnerHTML``` and thus React nodes are not valid placeholder values.

## Extraction
Using one of the functions above will mark a string for extraction into a standard gettext PO file. To extract strings, run `gulp extract`. You must pass strings directly into the functions for them to be captured.

## Messages
A generated PO file is not used directly by server or client, and must first be converted into JSON. To generate messages for the client/server, run `gulp messages`.

## Plurality
Gettext utilize a language's plural rule, which varies wildly. As an example, for the snippet above:

| Locale | friendCount | Output |
| --- | --- | --- |
| en-US | 0 | Facebook (0) friends |
|  | 1 | Facebook (1) friend |
|  | 2 | Facebook (2) friends |
| fr-FR | 0 | Facebook (0) ami |
|  | 1 | Facebook (1) ami |
|  | 2 | Facebook (2) amis |

Notice that for the French language, the plural rules is different from English. In some languages there is no distinction between singular and plural, and for others there are 6 plural forms with special rules.

## Best Practices
1. **Extract only complete strings**

  The meaning of complete is hard to establish & finding the right balance can be difficult.

    * At the very least, do not ever split words like `Bird` and `(s)` (should be using plural tools anyway).
    * Do not split a sentence  like `Here are some things:` `one thing, second thing`.
    * Splitting a paragraph into sentences can be okay if the sentences do not depend on one another (or give context to one another).
2. **Use context as necessary**

  Some strings require more information about the context they are used in. For example, the string `Flag` can mean different things (one being literally a flag, another being 'to report' a comment for example). Context allows the same string 'Flag' to have different translations. 

  Although not the intention of context, you can use it to give small comments to the translator, such as about where a fragment belongs. Do your best to avoid this use case though.
3. **Interpolate only with the provided tools**

  Do not concatenate user strings with `+` operator or templating tools from ES6. Rely solely on the provided components, or if doing string interpolation, use `interpolate` from the `Interpolator` module.
4. **Re-use existing strings**

  Adding a string that differs only slightly adds little to no value but adds potential wait time for the translation.
5. **Avoid HTML as much as possible**
  
  Injecting a string with HTML means going against React best practices. It also means making it more difficult for the translator. For example, it is hard to understand that `<div class='x'>Hello</div><div class='x'>World</div>` is style split the words into two lines. This is not often possible, but may be necessary to keep a string fully intact. 

6. **Do not add strings to style sheets**

  CSS does not allow extraction of strings. Avoid the `content` CSS property unless it's for symbols - even then it may not be a good idea as one symbol can mean something else in another language.

7. **Avoid certain text transformations**

  The meaning of a string in one form (e.g. fully capitalized) may not be appropriate in another language. If your strings needs to be fully capitalized, it must be extracted as such so that translators can change the letter case. Avoid transformation through methods like `toUpperCase()` and `text-transform: uppercase;` which do not even work for certain characters.
