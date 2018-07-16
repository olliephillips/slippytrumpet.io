+++
title = "Sett, a BadgerDB abstraction"
description = "A little Go package to make BadgerDB easier (for me) to work with"
date = "2018-05-04T10:08:30+01:00"

tags = [
	"golang",
    "badgerdb",
]

categories = [
    "programming",
]
+++


## A little Go package to make BadgerDB easier (for me) to work with

I've noticed that as an "older" developer, often, one of the first things I'll do with a new package/API I'm using, is abstract it into just the bits I need and/or that my cognitive resources can cope with.  

The end result is often something akin to plain english (in terms of code), and while I'm not sure this says much for my cognition and memory, I do often wonder why this is not better syntax full stop?

Here's an example. This package, named [Sett](https://github.com/olliephillips/sett), abstracts the [BadgerDB](https://github.com/dgraph-io/badger) API, the focus simply on easy reuse of the BadgerDB methods that I'm likely to want to use most frequently.

I'm not suggesting there's anything wrong with the BadgerDB API, only that, for me, the syntax seems quite complex and is therefore not easy to recall. Hence me writing Sett which literally just hides the complicated code enabling me to be more productive with BadgerDB.

To give one example, below we retrieve the value of a single key  from the data store and print it. First with the BadgerDB method, and then with Sett.

### Get with BadgerDB
```
// BadgerDB
var answer []byte
err := db.View(func(txn *badger.Txn) error {
  item, err := txn.Get([]byte("question"))
  if err != nil {
    return err
  }
  answer, err := item.Value()
  if err != nil {
    return err
  }
  return nil
})
if err != nil {
	log.Fatal(err)
}
fmt.Printf("The answer is: %s\n", answer)
```
<br/>
### Get with Sett
```
// Sett
answer, err := db.Get("question")
if err != nil {
	log.Fatal(err)
}
fmt.Printf("The answer is: %s\n", answer)
```
<br/>
Which version can you remember?

I should however point out that in BadgerDB, ```db.View``` is a read transaction wrapper.  So, while the above examples both include a single ```Get``` transaction, BadgerDB could accomodate more. 

We could for example retrieve a second key in the same transaction, which would doubtless be more efficient than the two sequential ```db.View``` transactions, required to get two keys when using Sett. 

But, realistically, how often will I want more than one key, selected by specific key? Should I find the need, a small modification to the Sett API could facilitate the return of multples keys in the one ```GET``` call. 

## "Virtual" tables

One feature of the Sett API that I find indispensible, is virtual tables. 

Tables are effectively created by adding a prefix to the key which is stored in BadgerDB, hence virtual, since tables are not a feature of BadgerDB.

With an interface formalised through the Sett API, I've found it much easier to reason about the data I'm working with, by thinking in terms of tables.

Here's an example which shows how the use of these virtual tables allows key reuse. 

```
s.Table("client").Set("1234", "client data")
// real key is "client1234"
s.Table("client").Get("1234")
// returns "client data"

s.Table("supplier").Set("1234", "supplier data")
// real key is "supplier1234"
s.Table("supplier").Get("1234")
// returns "supplier data"
```
<br/>
Tables also allow us to implement other nice features such as ```Drop()``` which, as you might expect, deletes the virtual table or more accurately, all keys with the table prefix.

## Updates
BadgerDB also includes a ```db.Update``` transaction wrapper designed to wrap ```Set``` and ```Delete``` transactions, and it's possible to write or delete multiple items by key. 

This is definitely something I want to do efficiently via Sett, so I implemented batch updates - and the API is not dissimilar to the functionality that used to exist in BadgerDB itself!


Items are added to the dataset to be stored using ``Batchup()```.

Large datasets are split into smaller batches of 500 (by default) and each batch of 500 is passed to a goroutine tasked with inserting those 500 keys. We can achieve very high write speed using concurrency and goroutines, though optimum batchsize will depend on size of the dataset and the performance of your hardware.

Sett splits the dataset into batches for you, in the background, no need to write your own goroutines.

Here's a very simple example which creates a batch of three keys/values, before submitting the batch to BadgerDB for insertion to the "client" table.

```
s.Batchup("hello", "world")
s.Batchup("hello-again", "world")
s.Batchup("goodbye", "world")

s.Table("client").SetBatch()
```
<br/>
There's more to Sett, and the README on Github covers it.  

And there's room to improve it. One area of concern is reporting and returning from goroutines which error, and this is something I'll be looking into.

If you find Sett useful, let me know in the comments. If I've made any errors please also let me know. I'm always learning.