## Use Positive Identifiers

Or _remove double negative_.

## Bad code

```js
var idIsNotUnique;

// Check: do you consider following two lines as readable?
idIsNotUnique = true;
idIsNotUnique = false;

// We need to branch if ID is unique.
if (!idIsNotUnique) {
  // Check: how much time have you spend reading this condition?
}
```

## Good code

```js
var idIsUnique;

idIsUnique = true;
idIsUnique = false;

if (idIsUnique) {
  // Check: is it quicker to read then `if (!idIsNotUnique)` written above?
}

// Teting for non-unique ID remains readable.
if (!idIsUnique) {
}
```

## Double negative

* http://www.refactoring.com/catalog/removeDoubleNegative.html
* https://en.wikipedia.org/wiki/Double_negation
