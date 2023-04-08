# Mutex

Being single-threaded doesn't necessarily mean we wouldn't eventually end up in some kind of race conditions. JavaScript is single-threaded but there is no race condition only for language implementers their self. They devide execution time for different sections of code and end user practically experience some kind of multi-threaded behavior.
We some how have to address this kind of uncertainty that we couldn't be sure in every situation which part of our code is going to run next. Mutex aims to provide this necessity for JavaScript developers which is really needed in many different situations.

## What is Mutex?
Mutex is a short for **Mut**ual **Ex**clusion. Freely speaking, mutual exclution is just a flag. It signals other threads do not modify or relay on data marked by the flag at current time. If a thread needs to change or have to be assured the data is not going to change simultaneously by other threads it has to acquire reponsibility for that flag or wait in a queue until the flag is available to acquire. It is important to know that mutual exclution does not prevent any unwanted change to the data. It only guides working threads to prevent conflictions with each other. However mutual exclusion can be used internally to prevent any unwanted data modification from outer scope.

For more information please refer to Wikipedia page on [Mutual Exclution](https://en.wikipedia.org/wiki/Mutual_exclusion)

## How does it work?
First of all what does it mean to have mutual exclution in a single-threaded environment? If there is no other thread running who is supposed to enter in a race condition with your code? It is simple. You your self. Infact JavaScript language design is trapping you in some kind of race conditions. Behind the scenes control is shifting between different parts of code constantly and sometimes triggers from outside world like file system, timers, networking, or other parts of operating system are received. JavaScript is doing everything very fast and it appears concurrent, but if you block the only working thread the language have, you are disabling its ability to shift control and do other things.

What you expect from following:

```js
const startTime = Date.now();

setTimeout(() => {
  const endTime  = Date.now();
  const duration = Math.floor((endTime - startTime) / 1000);

  console.log(`This message is set to be displayed after 1 second but it displayed after ${duration} seconds!`);
}, 1000);

while (true) {
  const endTime  = Date.now();
  const duration = Math.floor((endTime - startTime) / 1000);

  if (duration >= 3) {
    break;
  }
}
```
```
This message is set to be displayed after 1 second but it displayed after 3 seconds!
```

Now let's change it a bit:

```js
const wasteCPUCyclesInSeconds = (seconds) => {
  const startTime = Date.now();

  while (true) {
    const endTime  = Date.now();
    const duration = Math.floor((endTime - startTime) / 1000);

    if (duration >= seconds) {
      break;
    }
  }
}

const startTime = Date.now();
let   stop      = false;

setTimeout(() => {
  const endTime  = Date.now();
  const duration = Math.floor((endTime - startTime) / 1000);

  stop = true;

  console.log(`This message is set to be displayed after 1 second but it displayed after ${duration} seconds!`);
  console.log(`  stop: ${stop}`);
}, 1000);

console.log('Wasting CPU cycles for 2 seconds...');
wasteCPUCyclesInSeconds(2);

if (!stop) {
  console.log('Wasting CPU cycles for 1 second...');
  wasteCPUCyclesInSeconds(1);
}

console.log(`  stop: ${stop}`);
```
```
Wasting CPU cycles for 2 seconds...
Wasting CPU cycles for 1 second...
  stop: false
This message is set to be displayed after 1 second but it displayed after 3 seconds!
  stop: true
```

You see. You wanted to prevent wasting CPU cycles for another 1 second but the only thread running by JavaScript was busy.

Now let's switch to `async` without `await`:

```js
(async () => {
  const wasteCPUCyclesInSeconds = async (seconds) => {
    ...
    
    console.log(` Finished after ${seconds} seconds.`);
  }

  .
  .
  .

  console.log('Wasting CPU cycles for 2 seconds...');
  wasteCPUCyclesInSeconds(2);

  if (!stop) {
    console.log('Wasting CPU cycles for 1 second...');
    wasteCPUCyclesInSeconds(1);
  }

  console.log(`  stop: ${stop}`);
})();
```
```
Wasting CPU cycles for 2 seconds...
 Finished after 2 seconds.
Wasting CPU cycles for 1 second...
 Finished after 1 seconds.
  stop: false
This message is set to be displayed after 1 second but it displayed after 3 seconds!
  stop: true
```

Still the same. But what happened here? `wasteCPUCyclesInSeconds(2);` and `wasteCPUCyclesInSeconds(1);` were called asynchronously! It's because the only available thread have to enter asynchronous `wasteCPUCyclesInSeconds` and it is caught there until it finishes executing the method then it can continue on caller code.

Now let's use `Promise`:

```js
const wasteCPUCyclesInSeconds = (seconds) => {
  return new Promise(resolve => {
    const startTime = Date.now();

    while (true) {
      const endTime  = Date.now();
      const duration = Math.floor((endTime - startTime) / 1000);

      if (duration >= seconds) {
        break;
      }
    }

    console.log(` Finished after ${seconds} seconds.`);

    resolve();
  });
}

.
.
.

console.log('Wasting CPU cycles for 2 seconds...');
wasteCPUCyclesInSeconds(2);

if (!stop) {
  console.log('Wasting CPU cycles for 1 second...');
  wasteCPUCyclesInSeconds(1);
}

console.log(`  stop: ${stop}`);
```
```
Wasting CPU cycles for 2 seconds...
 Finished after 2 seconds.
Wasting CPU cycles for 1 second...
 Finished after 1 seconds.
  stop: false
This message is set to be displayed after 1 second but it displayed after 3 seconds!
  stop: true
```

Nothing changed. It does not matter you wait for `wasteCPUCyclesInSeconds` to complete or you don't. It can not. JavaScript with its only thread is unable to escape a blocking code.

Who in the world is using a blocking code in JavaScript? Maybe it is rare that someone wants something like what we have created here, but it is actually occuring more than often in our codes. Every time we are running a long running calculation of data we are blocking. For example processing a large dataset. When we are blocking, JavaScript is completely blinded about what is happening elsewhere. And in case of Node.js or other frameworks and environments outside browser, we are no anymore running JavaScript in a tab in our favorite browser. It is a huge drawback for many type of applications if it can not run CPU intensive works without blocking.

So, what is solution? How we can survive from single tasking blackhole in 2020s? What we have is a single thread running our code and it is not directly in our hands too. Perfect solution is not possible for us. The language designers and its implementers have to reconsider what is best for JavaScript and its huge and growing ecosystem.

Back in 1990s when designing JavaScript it was enough for a scripting language running in an ancient browser to run in a single thread and later on it was a genius idea to run in a single-threaded environment to get ride of multi-threading bottlenecks. But JavaScript grew much beyond expectations and it stepped out of browser. Also in browser JavaScript is now beyond a simple scripting language. Maybe its simplicity and its single-threaded environment which freed up so many headaches in programming are main factors for its success, but today if the language wants to support its vast sociaty and ecosystem it has to have a solution for running blocking code and still being able to do other things. Changing principal design of the language and be a multi-threaded language has huge consequences. This is not our only option. The language can have a multi-threaded like mode which is explicitly requested by programmer. Nothing needs to be changed but introducing another syntax and feature.

For example like marking a method as asynchronous with `async` it can also be marked as non-blocking which means every other code can be executed while flow of control entered this method. That way the programmer its self is responsible for every possible race condition that may happen. With this approach still the language can enjoy its atomic like behavior on changing and manipulating data. It just magically do not block its self in an ancient solved programming problem.

For now, let's mimic what JavaScript can have that everyone be still proud of it.

First we somehow need to keep hands off flow of control so JavaScript can run any other blocked code. It is obvious it is impossible for us to implement this in synchronous context. so our only option is asynchronous context. Shifting flow of control means exiting from our running code but we have to be able to return to our code again when JavaScript was finished running postponed codes. So our best option is `Promise` if we don't want to introduce other obstacles and use callbacks. How we keep our hands off flow of control it's just a simple already expired timeout.

Here is what we have:

```js
new Promise((resolve) => setTimeout(resolve, 0))
```

For now, let's test our hands off approach:

```js
(async () => {
  const wasteCPUCyclesInSeconds = async (seconds) => {
    const startTime = Date.now();

    while (true) {
      const endTime  = Date.now();
      const duration = Math.floor((endTime - startTime) / 1000);

      if (duration >= seconds) {
        break;
      }
    }

    console.log(` Finished after ${seconds} seconds.`);
  }

  const startTime = Date.now();
  let   stop      = false;

  setTimeout(() => {
    const endTime  = Date.now();
    const duration = Math.floor((endTime - startTime) / 1000);

    stop = true;

    if (duration !== 1) {
      console.log(`This message is set to be displayed after 1 second but it displayed after ${duration} seconds!`);
    } else {
      console.log(`This message is set to be displayed after 1 second and it did display after 1 second!`);
    }

    console.log(`  stop: ${stop}`);
  }, 1000);

  console.log('Wasting CPU cycles for 2 seconds...');
  await wasteCPUCyclesInSeconds(2);

  await new Promise((resolve) => setTimeout(resolve, 0));

  if (!stop) {
    console.log('Wasting CPU cycles for 1 second...');
    await wasteCPUCyclesInSeconds(1);
  }

  console.log(`  stop: ${stop}`);
})();
```
```
Wasting CPU cycles for 2 seconds...
This message is set to be displayed after 1 second and it did display after 1 second!
  stop: true
 Finished after 2 seconds.
  stop: true
```

We are amazing. We did it only with one line of code. But there are two issues with our code:

1. `wasteCPUCyclesInSeconds(2);` is still blocking.
   - ...   
2. We introduced possibility of race condition almost like what multi-threaded languages have.
   - It is almost because we still have atomic actions in our critical section.




Uploading soon...
