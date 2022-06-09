---
layout: post
title: "Motivating Higher-Order Functions"
---

This post is a simple effort to show someone who has never used Higher-Order Functions why they are useful and valuable, and not just unnecessary complexity.

A Simple Problem
---

Let's say we've got the following code:

```
function doStuff() {
    let intermediate_result = doExpensiveOperationOne();
    let final_result = doExpensiveOperationTwo(intermediate_result);
    return final_result;
}
```

It's doing what it's supposed to do, but it's just too slow! We want to understand what is taking all of that time, so we do something simple and just count the number of seconds that it takes by checking the clock before and after we do our operations:

```
function doStuff() {
    let timestamp = (new Date()).getTime();
    let intermediate_result = doExpensiveOperationOne();
    let final_result = doExpensiveOperationTwo(intermediate_result);
    let time_elapsed = (new Date()).getTime() - timestamp;
    console.log(time_elapsed + " milliseconds have passed");
    return final_result;
}
```

This is helpful, but we realize quickly that we need to know which of these functions took how long, so we go even further; for the sake of making this realistic, we also build a helper function for getting the current time:

```
function getTimestamp() {
    return (new Date()).getTime();
}

function doStuff() {
    let timestamp_1 = getTimestamp();
    let intermediate_result = doExpensiveOperationOne();
    let timestamp_2 = getTimestamp();
    let final_result = doExpensiveOperationTwo(intermediate_result);
    let timestamp_3 = getTimestamp();
    time_for_operation_one = timestamp_2 - timestamp_1;
    time_for_operation_two = timestamp_3 - timestamp_2;
    console.log("Time for operation one: " + time_for_operation_one + "ms");
    console.log("Time for operation two: " + time_for_operation_two + "ms");
    return final_result;
}
```

This solves our problem, but what a nightmare! Our code is only "doing" 2 operations and returning the result -- 3 lines -- but we have SEVEN lines of code measuring the performance! And what's worse, we can't even group it together, because the timestamps HAVE to be gathered in between the function calls. How could we improve this?

Encapsulating
---

Instead of doing all of this bookkeeping manually around our function calls, what if we could encapsulate the timing within the function call? Something like this:

```
function getTimestamp() {
    return (new Date()).getTime();
}

function doExpensiveOperationOneTimed() {
    let timestamp_1 = getTimestamp();
    let result = doExpensiveOperationOne();
    let timestamp_2 = getTimestamp();
    let time_elapsed = timestamp_2 - timestamp_1;
    console.log("ExpensiveOperationOne executed in " + time_elapsed + " ms");
}

function doExpensiveOperationTwoTimed(arg) {
    let timestamp_1 = getTimestamp();
    let result = doExpensiveOperationTwo(arg);
    let timestamp_2 = getTimestamp();
    let time_elapsed = timestamp_2 - timestamp_1;
    console.log("ExpensiveOperationTwo executed in " + time_elapsed + " ms");
}

function doStuff() {
    let intermediate_result = doExpensiveOperationOneTimed();
    let final_result = doExpensiveOperationTwoTimed(intermediate_result);
    return final_result;
}
```

What an improvement! Our doStuff function is actually readable again!

But just like we filled doStuff with a bunch of boilerplate before, we've now filled each of our sub-functions with boilerplate too. This is kind of okay when it's this simple, but if we had 15 expensive operations chained together, making changes to how we time things would get really annoying and error-prone. For example, if we wanted to stop logging these timestamps, we would have to edit 15 functions.

Can we get rid of the duplication altogether?

Higher-Order Functions to the Rescue
---

We can deduplicate this too! All our "timing" sections did was take a timestamp before and after running a function. So what if we write a function that takes another function, and runs that other function between timestamp checks? Take a leap of faith with me:


```
function getTimestamp() {
    return (new Date()).getTime();
}

function timedWrapper(func) {
    // A function with ...args can take any number of arguments
    let wrapped_func = (...args) {
        let timestamp_1 = getTimestamp();
        func(...args);
        let timestamp_2 = getTimestamp();
        let time_elapsed = timestamp_2 - timestamp_1;
        console.log(func.name + " executed in " + time_elapsed + "ms");
    }
    return wrapped_func;
}

function doStuff() {
    let intermediate_result = timedWrapper(doExpensiveOperationOne)(); // We wrap the function BEFORE we call it
    let final_result = timedWrapper(doExpensiveOperationTwo)(intermediate_result);
    return final_result;
}
```

Now, this may seem a bit complicated at first, but there's some tremendous advantages to not duplicating any code, even if reading timedWrapper isn't trivial:

1. All updates are in one place, and all timing is using the same code. We can't forget to update something, and we can't accidentally have e.g. some code timing in milliseconds and some in seconds.
2. If we want to time a new function, we can do it trivially.
3. Once you understand this solution, it is easier to reason about. The nature of functions and wrappers means that I don't have to read how the timing is done to be confident it's not messing with what `doExpensiveOperationOne` and `doExpensiveOperationTwo` do.
