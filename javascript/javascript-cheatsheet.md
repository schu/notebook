# JavaScript Cheatsheet

Small JavaScript snippets.

## Augment objects in list of objects

```js
let l = [
  { text: "foo" },
  { text: "bar" }
];

l.map((obj, index) => ({ ...obj, index: index }))
// [ { text: 'foo', index: 0 }, { text: 'bar', index: 1 } ]

l.map((obj, index) => ({ ...obj, index: index, something: 'else' }))
// [
//   { text: 'foo', index: 0, something: 'else' },
//   { text: 'bar', index: 1, something: 'else' }
// ]
```
