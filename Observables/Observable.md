# Observable
An observable is like a promise that can resolve more than one value
Each time `next` is called, the observable resolves a value and pushes it to observers
Once `complete` is called, no more values can be returned (like in an iterator)
Observables can return values synchronously and asynchronously, depends on how you write it

## Compared to promises

All promises can technically be rewritten in terms of Observables, for example
```
let promise = new Promise( (resolve, reject) =>{
  try {
    resolve(doSomething());
  } catch (error) {
    reject(error);
  }
})

promise.then(value => doSomethingElse(value))

var observable = Rx.Observable.create(function (observer) {
  try {
    observer.next(doSomething())
  } catch (error) {
    observer.error(error);
  }
})

observable.subscribe(value => doSomethingElse(value))
```
Code looks pretty much the same, but use observer.next instead of resolve to push value.
And observer.error instead of reject to send error

## Hot and Cold

There are two types of observables: Hot and Cold

**Cold** Observables create their producer during subscription
**Hot** Observables use an outside producer for their subscription

**Producers** are a source of data that observables use to send values, i.e. DOM events, an interator, Web sockets, etc.

What this means is that cold observables won't produce data until something subscribes to it, since that is when the Producers is created. Hot observables start sending out data when they are created, even if there are no subscribers to "listen" for the data being sent

## Subscribing to Observables

We saw an example of this earlier `observable.subscribe(value => doSomething(value))`

An important thing to note is each instance of an observable will have its own setup

```
var s1 = observable.subscribe(value => {
  var x = 0;
  window.setInterval(() => {
    console.log(x++);
  },200)
})

var s2 = observable.subscribe(value => {
  var x = 0;
  window.setInterval(() => {
    console.log(x++);
  },200)
})
```

Here s1 and s2 will both output values every 200ms, but the values will be independent from each other. This means they have their own values of `x` and don't increment eachother

## Unsubscribing

Since observables can produce values indefinitely, there are situations where you want to stop it to save processing power, like in `s1` and `s2` above. To do that, you just need to set-up and unsubscribe function like so

```
var s1 = observable.subscribe(value => {
  var x = 0;
  var intervalId = window.setInterval(() => {
    console.log(x++);
  },200)
  
  return function unsubscribe() {
    clearInterval(intervalId);
  }
}) 
```

This is essentially like garbage collection in other languages. We have to do it manually for now, but in the future there might be libraries that abstract this work away.
