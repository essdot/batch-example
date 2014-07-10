# Batch

This class batches updates, then executes them using `requestAnimationFrame`. If `requestAnimationFrame` is not available, `setTimeout` will be used instead, with a 60fps timeout. When the frame occurs, all queued jobs will be executed in order.

Optionally, the constructor can be passed a `sync` argument, representing a custom sync function to be used instead of `requestAnimationFrame`.

## Usage

* **ctor([sync])**: This constructor optionally accepts a `sync` function, to be used in place of `requestAnimationFrame`. `sync` should accept a callback parameter, representing the job to be executed.

* **Batch.prototype.queue(fn)**: Add a job to the batch queue. The job will be executed in the next animation frame.  

* **Batch.prototype.add(fn)**: Returns a `queue` function. When `queue` is called, *fn* will be added to the batch queue, and when *fn* is executed, any arguments passed to `queue` will be passed on to *fn*.  

```javascript
var batch = new Batch();

var hello = function() { console.log('hello'); }
var hello2 = function() { console.log('hello again'); }

batch.queue(hello); 
batch.queue(hello2); 

// => hello
// => hello again

var log = function(s) { console.log(s); }

var q = batch.add(log);
q('xyz');
q('123');

// => xyz
// => 123

```

### Known Issues

* `Batch.prototype.add(fn)` will execute *fn* in the global context, not the context of the Batch object. Inside *fn*, `this` will refer to the global object.

* `Batch.prototype.queue(fn)` will fail to execute the jobs in the queue if the Batch object has a `sync` function defined.
