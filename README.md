```javascript
// One of the biggest challenges I've seen people face when using promises
// has to do with how to keep functions in a chain "pure". In other words, 
// how can functions in a promise chain have only one output for a unique 
// set of arguments, produce no side effects, and never rely on an external state?

// Let's start with an example of the problem

// Say we need to fire an http request to get a piece of information represented in json. 
// We have a function named getInfo() that returns a promise that resolves with the info needed.

return getInfo()
  .then(info => info);

// Now, we want to do two things with that info.
// 1. We want to store it somewhere (function storeInfo); and 
// 2. We want to transform it into a different model (function transformInfo).

return getInfo()
  .then(storeInfo)
  .then(transformInfo);

// We have a problem here, though. Let's assume storeInfo returns the result of a database query.
// To make this work, we'll have to do the following.

return getInfo()
  .then(info => storeInfo(info).then(() => info))
  .then(transformInfo);

// This isn't too bad, but it's also not ideal. In this scenario we are nesting promises twice.
// We can think of a scenario where the inner chain can go even further.
// It could quickly become hard to read. So we resort to the following
let myInfo;

return getInfo()
  .then(info => {
    myInfo = info;
    
    return storeInfo(info);
  })
  .then(() => transformInfo(myInfo));

// Here, we don't care about what storeInfo returns.
// We declare a variable at the upper scope, assign it in a step in the promise chain,
// and use it in subsequent steps.

// At this point, our functions are no longer pure. The last step depends on myInfo.
// Any way to get around this? Let's see.

getInfo()
  .then(info => Promise.all([info, storeInfo(info)]))
  .spread(transformInfo);

// What's going on here?
// In our first step, we're returning a promise that resolves to an array that contains a value (info) and the result of calling storeInfo with info.
// In our second and last step, we're using spread, which takes an array as the fulfillment value.
// And because transformInfo only cares about info, it just works.
// This assumes that transformInfo, of course, does't care about any other inputs.

// While the first step does have to know about storeInfo, it doesn't have to know about actual external states.
// It's not perfect, but it does make your chain look cleaner, more readable, and pure.
```
