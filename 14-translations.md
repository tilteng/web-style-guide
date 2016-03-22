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

These mimic standard gettext functions, and each one returns a function. The returned function must be invoked with a locale. This is done for you in `LocalizedMessage` which has a locale in the context, otherwise you must do it explicitly like so:
```javascript
const locale = ...;
<img alt={_('Hello')(locale)} />
```

## Translator Components
`LocalizedMessage` can accept placeholder values of any type, including React nodes.
```javascript
<LocalizedMessage timeAgo={<LocalizedTimeAgo date={...} />} message={_('You ate {timeAgo}!')} />
```

`LocalizedHTMLMessage` sets HTML directly using ```dangerouslySetInnerHTML``` and thus React nodes are not valid placeholder values.
```javascript
<LocalizedHTMLMessage name={'<span class="highlight">Bob</span>'} message={_('Hello {name}!')} />
``` 

Both components return a `span` as the parent element since React does not allow components to return a raw string.

## Extraction
Using one of the functions above will mark a string for extraction into a standard gettext PO file. To extract these marked strings, run `gulp extract`, which internally uses `xgettext`. You must pass strings directly as arguments into the functions for them to be marked for extraction.

## Client and Server Messages
A generated PO file is not used directly by server or client, and must first be converted into JSON. To generate messages for the client/server, run `gulp messages`. The client will only download the messages it needs for its locale.

## Plurality
Gettext utilizes a language's plural rules, which varies wildly. As an example, for the snippet above:

| Locale | friendCount | Output |
| --- | --- | --- |
| en-US | 0 | Facebook (0) friends |
|  | 1 | Facebook (1) friend |
|  | 2 | Facebook (2) friends |
| fr-FR | 0 | Facebook (0) ami |
|  | 1 | Facebook (1) ami |
|  | 2 | Facebook (2) amis |

Notice that for the French language (as a speaker from France), the plurality is slightly different from English. In some languages there is no distinction between singular and plural, and for others there can be as many as 6 plural forms with special rules. 

As a developer, you only have to worry about the plural rule in the English (U.S.) language, but you should also keep in mind that you might need to use plural translators `_n` or `_nc` even if the singular form is equivalent to the plural in the English language.

## Best Practices
1. **Extract only complete strings**

  The meaning of complete is hard to establish & finding the right balance can be difficult.

    * At the very least, do not ever split words like `Bird` and `(s)` (this example should be using plural tools anyway).
    * Do not split a sentence like `Here are some things:` `one thing, second thing.`.
    * Splitting blocks of text into individual sentences can be fine if the sentences do not depend on one another (or give context to one another). This is not guaranteed to always be true, but it is a middle ground that should satisfy both development and translation.
2. **Use context as necessary**

  Some strings require more information about the context they are used in. For example, the string `Flag` can mean different things (one being literally a flag, another being 'to report' a comment for example). Context allows the same string 'Flag' to have different translations. 

  Although not the intention of context, you can also use it to give small comments to the translator, such as about where a fragment belongs. Do your best to avoid this use case though.
3. **Interpolate only with the provided tools**

  Do not concatenate user strings with `+` operator or templating tools from ES6. Rely solely on the provided components, or if doing string interpolation, use `interpolate` from the `Interpolator` module.
4. **Re-use existing strings**

  Adding a string that differs only slightly adds little to no value but adds potential wait time for the translation to come back.
5. **Avoid HTML as much as possible**
  
  Injecting a string with HTML means going against React best practices. It also means making it more difficult for the translator. For example, it is hard to understand that `<div class="x">Hello</div><div class="x">World</div>` can be splitting the words into two lines through CSS. However, avoiding HTML is sometimes not possible, and may be necessary to keep a string fully intact.

  At the very least, you should not be adding unnecessary HTML to the strings marked for translation, such as `<p>Hello World</p>`.

6. **Do not add strings to style sheets**

  CSS does not allow extraction of strings. Avoid the `content` CSS property unless it's for symbols - even then it may not be a good idea as one symbol can mean something else in another language.

7. **Avoid certain text transformations**

  The meaning of a string in one form (e.g. fully capitalized) may not be appropriate in another language. If your string needs to be fully capitalized, it must be extracted as such so that translators can change the case appropriately. Avoid text transformation through methods like `toUpperCase()` and `text-transform: uppercase;` which do not even work for certain characters.
  
8. **Consider string size**

  This should be done early in the design process, but take the extra step to verify that your styling accomodates longer text length. 300% expansion from English is not uncommon in certain languages.
