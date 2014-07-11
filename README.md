# Batch

This class batches jobs, then executes them using `requestAnimationFrame`. (If `requestAnimationFrame` is not available, `setTimeout` will be used instead, with a 60fps timeout.) The jobs are queued, and executed in order.

Optionally, the constructor can be passed a `sync` argument, representing a custom sync function to be used instead of `requestAnimationFrame`.

## Usage

* **ctor([sync])**: This constructor optionally accepts a `sync` function, to be used in place of `requestAnimationFrame`. `sync` should accept a callback parameter, representing the job to be executed.

* **Batch.prototype.queue(fn)**: Enqueue a job. `fn` will be executed before the next animation frame.  

```javascript
var batch = new Batch();

var hello = function() { console.log('hello'); }
var hello2 = function() { console.log('hello again'); }

batch.queue(hello); 
batch.queue(hello2); 

// => hello
// => hello again
```

* **Batch.prototype.add(fn)**: Returns a `queue` function. When this function is called, `fn` will be enqueued. When `fn` is executed, any arguments passed to `queue` will be passed on to `fn`.  

```javascript
var batch = new Batch();

var logCar = function(car) { console.log('My car is a ' + car + '.'); }
var logHouse = function(house) { console.log('My house is a ' + house + '.'); }

var qCar = batch.add(logCar);
var qHouse = batch.add(logHouse);

qCar('Toyota Corolla');
qHouse('small bungalow');

// => My car is a Toyota Corolla.
// => My house is a small bungalow.

qCar('Peugot');
// => My car is a Peugot.
```

### Known Issues

* Jobs added with the `queue` function returned by `Batch.prototype.add(fn)` will execute in the global context, not the context of the Batch object. Inside `fn`, `this` will refer to the global object. To work around this, call the `queue` function returned by `Batch.prototype.add(fn)` in the context you would like `fn` to have:

```javascript
var batch = new Batch();

var coolItem = { isCool: true };
var isItCool = function() { 
	if(this.isCool === true) {
		console.log("Yep, this is cool."); 
	} else {
		console.log("Nope, this ain't cool."); 
	}
};

var qCool = batch.add(isItCool);

qCool();

// => Nope, this ain't cool.

qCool.call(coolItem);

// => Yep, this is cool.
```

* An instance of the `queue` function returned by `Batch.prototype.add(fn)` can only add one job per frame. If this function is called more than once per frame, the last invocation wins and the previous invocations do nothing. When the job is executing, `fn` will receive the parameters passed to the most-recent invocation of `queue`:

```javascript
qCool.call(coolItem); qCool.call(coolItem); qCool();

// => Nope, this ain't cool.

qCar('Audi TT'); qCar('Ford Pinto');

// => My car is a Ford Pinto.
```

* If the Batch object has a `sync` function defined, any jobs in the queue will not be executed. The `sync` function will never be invoked.