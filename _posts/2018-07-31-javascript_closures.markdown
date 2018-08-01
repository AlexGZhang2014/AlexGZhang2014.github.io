---
layout: post
title:      "Javascript Closures"
date:       2018-08-01 00:44:16 +0000
permalink:  javascript_closures
---


What is a closure? In the context of the real world, closure is the act of bringing something to an end and accepting that so one can move onto new things. In the context of JavaScript programming, however, closure means something different entirely! A closure in programming is an attribute of a function such that when the function is declared, it holds onto not just its own scope, but the scope of the environment it was created in. This means that functions can be declared in one place and then be executed in another place and still be able to use variables that are not explicitly declared in the second environment.

Let's give an overly contrived example so that all of this makes sense:
```
function generatorOfFunctions(variable) {
  return function(otherVariable) {
    return `${otherVariable} is part of this function's scope, but ${variable} is part of the outer scope this function was declared in!`
  }
}

const firstFunction = generatorOfFunctions(77);

firstFunction("ALLCAPS")
// "ALLCAPS is part of this function's scope, but 77 is part of the outer scope this function was declared in!"
```

In the above example, the first function "generatorOfFunctions" gives a return value of another function. That returned function returns a string that contains both its passed-in argument and the argument from the outer function. We stored the returned function in a const variable "firstFunction" and then invoked that stored function with its own argument. This stored function was able to output the string using both its own argument and the argument from the outer scope because of the use of a closure. Because "variable" was part of the scope that the 2nd function was declared in (i.e. the first function), it is able to be accessed and used by the 2nd function no matter where else the 2nd function is invoked.

So if we were to declare another function and then invoke it:
```
const secondFunction = generatorOfFunctions("Another variable");

secondFunction("Different variable")
// "Different variable is part of this function's scope, but Another variable is part of the outer scope this function was declared in!"
```
we would still see the expected return value - a string that includes both arguments. Closures are wonderful because they allow us to create functions like this that can be used elsewhere. It's amazing how functions "remember" the scope that they were declared in and are able to then use any variable that was declared in that outer scope. There are many more topics in JavaScript to learn, but I just wanted to give a quick summary about closures because they are so cool!

