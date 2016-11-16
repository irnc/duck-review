# Idiomatic Ramda

## Misused Ramda

```js
const categories = R.map(
  category => category.toLowerCase(),
  R.filter(R.identity, R.uniq(R.flatten(R.pluck('categories')(R.values(products)))))
);
```

### Imperative Way of Thought

TK

## Functional Way of Thought

TK

### Rethought Ramda Usage

## References

- TODO try runkit for duck review https://runkit.com/irnc/582c3ca74e112e0014358040
