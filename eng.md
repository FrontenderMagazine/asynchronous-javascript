<section class="post-content">![asynchronous-javascript-cover][1] 

The `async` functions are just around the corner - but the journey to here was
quite long. Not too long ago we just wrote[callbacks][2], then the Promise/A+
specification emerged followed by[generator functions][3] and now the async
functions.

Let's take a look back and see how asynchronous JavaScript evolved over the
years.

## Callbacks {#callbacks}

*It all started with the [callbacks][2].* 

### Asynchronous JavaScript {#asynchronousjavascript}

Asynchronous programming, as we know now in JavaScript, can only be achieved
with functions being first-class citizens of the language: they can be passed 
around like any other variable to other functions. This is how callbacks were 
born: if you pass a function to another function*(a.k.a. **higher order
function**)* as a parameter, within the function you can call it when you are
finished with your job. No return values, only calling another function with the
values.

    Something.save(function(err) {  
      if (err)  {
        //error handling
        return;
      }
      console.log('success');
    });
    

These so called **error-first callbacks** are in the heart of Node.js itself -
the core modules are using it as well as most of the modules found on NPM.

The challenges with callbacks:

*   it is easy to build callback hells or spaghetti code with them if not used
    properly
   
*   error handling is easy to miss
*   can't return values with the `return` statement, nor can use the `throw`
    keyword
   

Mostly because of these points the JavaScript world started to look for
solutions that can make asynchronous JavaScript development easier.

One of the answers was the [async][4] module. If you worked a lot with
callbacks, you know how complicated it can get to run things in parallel, 
sequentially or even mapping arrays using asynchronous functions. Then the async
module was born thanks to[Caolan McMahon][5].

With async, you can easily do things like:

    async.map([1, 2, 3], AsyncSquaringLibrary.square,  
      function(err, result){
      // result will be [1, 4, 9]
    });
    

Still, it is not that easy to read nor to write - so comes the Promises.

## Promises {#promises}

The current JavaScript Promise specifications date back to 2012 and available
from ES6 - however Promises were not invented by the JavaScript community. The 
term comes from[Daniel P. Friedman][6] from 1976. 

**A promise represents the eventual result of an asynchronous operation.**

The previous example with Promises may look like this:

    Something.save()  
      .then(function() {
        console.log('success');
      })
      .catch(function() {
        //error handling
      })
    

You can notice that of course Promises utilize callbacks as well. Both the 
`then` and the `catch` registers callbacks that will be invoked with either the
result of the asynchronous operation or with the reason why it could not be 
fulfilled. Another great thing of Promises is that they can be chained:

    saveSomething()  
      .then(updateOtherthing)
      .then(deleteStuff)  
      .then(logResults);
    

When using Promises you may have to use polyfills in runtimes that don't have
it yet. A popular choice in these cases is to use[bluebird][7]. These libraries
may provide a lot more functionality than the native one - even in these cases
**limit yourself to the features provided by Promises/A+ specifications**.

But why shouldn't you use the sugar methods? Read 
[Promises: The Extension Problem][8]. For more information on Promises, refer
to the[Promises/A+ specification][9].

*You may ask: how can I use Promises when most of the libraries out there
exposes a callback interfaces only?*

Well, it is pretty easy - the only thing that you have to do is wrapping the
callback the original function call with a Promise, like this:

    function saveToTheDb(value) {  
      return new Promise(function(resolve, reject) {
        db.values.insert(value, function(err, user) { // remember error first ;)
          if (err) {
            return reject(err); // don't forget to return here
          }
          resolve(user);
        })
      }
    }
    

Some libraries/frameworks out there already support both, providing a callback
and a Promise interface at the same time. If you build a library today, it is a 
good practice to support both. You can easily do so with something like this:

    function foo(cb) {  
      if (cb) {
        return cb();
      }
      return new Promise(function (resolve, reject) {
    
      });
    }
    

Or even simpler, you can choose to start with a Promise-only interface and
provide backward compatibility with tools like[callbackify][10]. Callbackify
basically does the same thing that the previous code snippet shows, but in a 
more general way.

## Generators / yield {#generatorsyield}

[JavaScript Generators][11] is a relatively new concept, they were introduced
in ES6*(also known as ES2015)*.

> Wouldn't it be nice, that when you execute your function, you could pause it
> at any point, calculate something else, do other things, and then return to it, 
> even with some value and continue?
>

This is exactly what generator functions do for you. When we call a generator
function it doesn't start running, we will have to iterate through it manually.

    function* foo () {  
      var index = 0;
      while (index < 2) {
        yield index++;
      }
    }
    var bar =  foo();
    
    console.log(bar.next());    // { value: 0, done: false }  
    console.log(bar.next());    // { value: 1, done: false }  
    console.log(bar.next());    // { value: undefined, done: true }  
    

If you want to use generators easily for writing asynchronous JavaScript, you
will need[co][12] as well.

> Co is a generator based control flow goodness for Node.js and the browser,
> using promises, letting you write non-blocking code in a nice-ish way.
>

With `co`, our previous examples may look something like this:

    co(function* (){  
      yield Something.save();
    }).then(function() {
      // success
    })
    .catch(function(err) {
      //error handling
    });
    

You may ask: what about operations running in parallel? The answer is simpler
than you may think*(under the hoods it is just a `Promise.all`)*:

    yield [Something.save(), Otherthing.save()];  
    

## Async / await {#asyncawait}

Async functions were introduced in ES7 - and currently only available using a
transpiler like[babel][13]. *(disclaimer: now we are talking about the `async`
keyword, not the async package
)*

In short, with the `async` keyword we can do what we are doing with the
combination of`co` and generators - except the hacking. 

![denicola-yield-await-asynchronous-javascript][14] 

Under the hood `async` functions using Promises - this is why the async
function will return with a`Promise`.

So if we want to do the same thing as in the previous examples, we may have to
rewrite our snippet to the following:

    async function save(Something) {  
      try {
        await Something.save()
      } catch (ex) {
        //error handling
      }
      console.log('success');
    } 
    

As you can see to use an async function you have to put the `async` keyword
before the function declaration. After that, you can use the`await` keyword
inside your newly created async function.

Running things in parallel with `async` functions is pretty similar to the 
`yield` approach - except now the `Promise.all` is not hidden, but you have to
call it:

    async function save(Something) {  
      await Promise.all[Something.save(), Otherthing.save()]
    } 
    

[Koa][15] already supports `async` functions, so you can try them out today
using`babel`.

    import koa from koa;  
    let app = koa();
    
    app.experimental = true;
    
    app.use(async function (){  
      this.body = await Promise.resolve('Hello Reader!')
    })
    
    app.listen(3000);  
    

## Further reading {#furtherreading}

Currently we are using [Hapi with generators][3] in production in most of our
new projects - alongside with[Koa as well][11].

> Which one do you prefer? Why? I would love to hear your comments!</section>

 [1]: img/the-evolution-of-asynchronous-javascript.png
 [2]: https://blog.risingstack.com/node-js-best-practices/

 [3]: https://blog.risingstack.com/hapi-on-steroids-using-generator-functions-with-hapi/
 [4]: https://www.npmjs.com/package/async
 [5]: https://twitter.com/caolan
 [6]: https://en.wikipedia.org/wiki/Daniel_P._Friedman
 [7]: https://github.com/petkaantonov/bluebird
 [8]: http://blog.getify.com/promises-part-4/
 [9]: https://promisesaplus.com/
 [10]: https://www.npmjs.com/package/callbackify
 [11]: https://blog.risingstack.com/introduction-to-koa-generators/
 [12]: https://www.npmjs.com/package/co
 [13]: http://babeljs.io
 [14]: img/denicola-yield-await-asynchronous-javascript.JPG
 [15]: http://koajs.com/