# evidence

Track the history of an object.

[![Build Status](http://img.shields.io/travis/fardog/evidence/master.svg?style=flat)](https://travis-ci.org/fardog/evidence)
[![npm install](http://img.shields.io/npm/dm/evidence.svg?style=flat)](https://www.npmjs.org/package/evidence)

## Example

At its simplest, you simply write a value to the history, and it's saved as the
latest state. Any previous values pushed further down the stack.

```javascript
var evidence = require('evidence')

var instance = evidence()

instance.write({item: 'One'})
instance.get(0) // {item: 'One'}
instance.write({item: 'Two'})
instance.get(1) // {item: 'Two'}
instance.length // 2
```

But being a stream, you can pipe data directly to it:

```javascript
var evidence = require('evidence')
  , stream = require('through')()

var instance = evidence()

stream.pipe(instance)
stream.write({item: 'One'})

instance.get(0) // {item: 'One'}
```

History is only saved if the last value written isn't the same as the item in
the top of the stack.

## API

- `evidence([size])` -> Duplex Stream: Instantiates a new instance, and returns
  a duplex stream that saves objects written to it into its internal state, and
  emits that state so long as it's changed.
    - `size` (Number): The number of objects that should be saved before older
      objects are discarded. Defaults to `100`.

### Methods and Properties

- `instance.get([index])` - Get the item saved at `index`, and get the last
  item written if `index` isn't provided. New items written are saved to the
  front of the stack, so index `0` is the latest, `1` is the item that was
  written prior to `0`, and so on.
- `instance.offset([index])` - Sets the new head element as the element at
  `index`. Any further `get` calls will act as though the element you set is
  `0`; then `1` will be the item saved prior to the new `0`, and so on.
- `instance.length` - The current length of the stack.
- `instance.currentOffset` - The current offset on the stack

### Events

- Emits a `data` event, like any stream. The value emitted is the last-written
  object.
- Emits a `truncated` event when any item is truncated from the stack due to
  the stack outgrowing the specified `size`, or due to a new state being saved
  when an offset has been set. Emitted with the event will be an array of the
  removed items.

## Notes

- When understanding this module, think about it in terms of undo/redo. If you
  write something to it, that's the latest undo state. If you `offset(1)` that
  would be like a "undo". Now the element that was at `1` is the new head of
  the stack, and the stack's length is now one less than it once was. If you
  want to redo, you can set `offset(-1)` to go back. Any "undone" data will be
  lost when writing new values to the stack, so `offset(1)` followed by a
  `write()` would lose the old element at position `0`.
- Writing to a stack that has been `offset()` will truncate any elements newer
  than `offset` when written to.
- Use of the ``length`` property will fail on IE8 and older (if using in the
  browser); if you need older browser compatibility, use
  `instance.getLength()`, which is provided for compatibility.

## License

MIT. See [LICENSE](./LICENSE) for details.
