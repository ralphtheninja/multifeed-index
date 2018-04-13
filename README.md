# multi-hypercore-index

> build an index over a set of hypercores

Traverses a set of hypercores (as a
[multi-hypercore](https://github.com/noffle/multi-hypercore)) and calls a user
indexing function to build an index.

## Example

Let's build a key-value index:

```js
var hypercore = require('hypercore')
var multicore = require('multi-hypercore')
var indexer = require('.')
var umkv = require('unordered-materialized-kv')
var ram = require('random-access-memory')
var memdb = require('memdb')

var multi = multicore(hypercore, ram, { valueEncoding: 'json' })

var kv = umkv(memdb())

var hyperkv = indexer({
  cores: multi,
  batch: function (nodes, next) {  // each 'nodes' entry has an 'id' field that is "hypercorekey@seq"
    kv.batch(nodes, next)
  }
})

function append (w, data, cb) {
  w.append(data, function (err) {
    if (err) return cb(err)
    var id = w.key.toString('hex') + '@' + (w.length - 1)
    cb(null, id)
  })
}

multi.writer(function (err, w) {
  append(w, {
    key: 'foo',
    value: 'bax',
    links: []
  }, function (err, id1) {
    console.log('id-1', id1)
    append(w, {
      key: 'foo',
      value: 'bax',
      links: [id1]
    }, function (err, id2) {
      console.log('id-2', id2)
      hyperkv.ready(function () {
        kv.get('foo', function (err, res) {
          if (err) throw err
          console.log(res)
        })
      })
    })
  })
})

```

outputs

```
id-1 9736a3ff7ae522ca80b7612fed5aefe8cfb40e0a43199174e47d78703abaa22f@0
id-2 9736a3ff7ae522ca80b7612fed5aefe8cfb40e0a43199174e47d78703abaa22f@1
[
  '9736a3ff7ae522ca80b7612fed5aefe8cfb40e0a43199174e47d78703abaa22f@1'
]
```

## API

...

## License

ISC
