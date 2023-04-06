# Mutex

Being single-threaded doesn't necessarily mean we wouldn't eventually end up in some kind of race conditions. JavaScript is single-threaded but there is no race condition only for language implementors their self. They devide execution time for different sections of code and end user practically experience some kind of multi-threaded behavior.
We some how have to address this kind of uncertainty that we couldn't be sure in every situation which part of our code is going to run next. Mutex aims to provide this necessity for JavaScript developers, especially for those who codes in Node.js which is really needed in many different situations.

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
  const wasteCPUCyclesInSeconds = async (seconds) => {
    ...
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
Wasting CPU cycles for 1 second...
  stop: false
This message is set to be displayed after 1 second but it displayed after 3 seconds!
  stop: true
```



Uploading soon...
