## Explainer

Errors - or Exceptions - can be swallowed within the callbacks used in the `*Observer` APIs. This work provides the user some control to capture and handle errors. It's intentionally designed to be direct and without changes to the error events.

### Expected Exceptions in the Observer API

Broadly speaking, there are three kinds of errors to be captured for the Observer model:

#### Configuration

Those are the exceptions thrown when you're setting up a new instance or setting elements to be observed. E.g. on the immediate call for `o.observe(elem)`. Those are immediate errors and can be be captured today.

Using the [ResizeObserver](https://drafts.csswg.org/resize-observer/#resize-observer-interface) as an example, we can illustrate this exception giving an invalid parameter as the callback function.

```js
new ResizeObserver({}); // TypeError
```

Users can handle this exception today, there is nothing to be done here.

#### <a id="effects"></a> Effects within the observable elements

Errors due to the resize effect of observable elements and the resize loop (over the boundaries, etc).

Some of these errors may already be recognized by each of Observer API's explainer. E.g. [ResizeObserver](https://github.com/WICG/resize-observer/blob/master/explainer.md#error-handling).

#### Callback executions

Execution errors of the ResizeObserver callback, when it's called. This seems like a little hazard bubbling out of the callback function execution.

One way to exemplify this would be an immediate throwing expression:

```js
new ResizeObserve(function() {
  throw new Error();
});
```

The error from the callback function cannot be captured from registering the elements through the `.observe` method. The registration does not imply a call to that function.

```js
const observer = new ResizeObserver(function() {
    throw new Error();
});

try {
    observer.observe(elem1);
} catch (e) {
    /* the only error to be captured here is if elem1 don't meet the parameters requirements */
}
```

The current Observer model has no control over the given error. This error would bubble out and eventually captured in the `window.onerror`. It would be useful for the Observer instances to provide some control to capture and/or handle the error.

While the `.onerror` remains the classic approach for handling errors in window and elements, it doesn't fit well in the Observer the same way it can't add an event listener as in `window.addEventListener`. This matches the idea on [observers API not being based in events](https://github.com/WICG/resize-observer/blob/master/explainer.md#why-an-observer-based-api-and-not-events).

This proposal tries a less complex solution that uses another callback for catching the errors.
​
```js
var observer = new ResizeObserver(function(entries) { /* ... */ });
observer.observe(elem1);
observer.observe(elem2);
observer.observe(elem3);

// The new proposed method
observer.catch(function(err) {
    /* error to be captured when the callback is executed */
});
```

This `catch` name is heavily inspired on the Promise's API but naming can be discussed and the current choice is not a hard requirement.

It's important to observe that registering the `.catch` callback can remove the bubbling out. It may work as an opt-in to ignore errors:

```js
observer.catch(function(err) { /* noop */ });
```

This method captures errors triggered in the callback function. E.g.:

```js
var observer = new ResizeObserver(function(entries) {
    throw new Error('meep');
});
observer.observe(elem1);
observer.observe(elem2);

// The new proposed method
observer.catch(function(err) {
    console.log(err.message); // 'meep'
});
```

### Open ended discussions and Variants

#### Multiple catch calls

What it means to call `.catch` more than once? 

```js
observer.catch(function(err) { console.log('1'); });
observer.catch(function(err) { console.log('2'); });
```

The current suggestion is for the observer to register every callback, instead of replacing them. This remains [similar to the Promise's `catch`](https://jsfiddle.net/2mpzoLrq/). The error object would be preserved for each call.

#### No definition (yet) to unregister catch callbacks.

This explainer does not define any way to unregister catch callbacks as there is little to no identified value in adding this so far.

#### Capturing the effects from observed elements

The catch callback can be used to capture errors triggered in the Observer callback but also expanded to error handling as described in the [`Effects within the observable elements`](#effects) section.

#### Setting the error callback in the constructor API

An alternative for the `.catch` method would be setting the error callback in the Observer constructor. E.g.:

```js
function fn(entries) { /* noop */ }
function errCallbackFn(err) { /* noop */ }

var observer = new ResizeObserver(fn, errCallbackFn);
```

Although, this would only allow registering one error callback and it gives bad intuition the error callback can be used the error is triggered by calling the regular callback function `fn`.
