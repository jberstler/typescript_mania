# Why are you reading this?
This is just my little journal of thoughts and questions as I try to wrap my head around TypeScript.

# What's up with `| undefined` typing?
I keep seeing TypeScript code where the explicit typing is something like `foo: SomeType | undefined`. This seems weird to me. Why not just declare this to be optional like `foo?: SomeType`? This is more succinct and, to my eyes, just clearer.

Let's break it down.

## Local Variables
There does not seem to be a way to declare  local variables as optional. That is, `let foo?: string` is a syntax error on the question mark. Ok, if that is the case, the only way to explicitly declare that a local variable is potentially undefined is to use `| undefined` typing, such as `let foo: string | undefined`

Ok, that one was easy.

## Class Properties
This one a little more difficult as class properties are allowed to use the `?` syntax to declare the property is optional. So what is the difference between, say, `foo?: string` and `foo: string | undefined` apart from the fact that the latter is more verbose and (to my eyes) less immediately clear.

Perhaps you can assign `null` to the optional property and not the `| undefined` property? Nope. In fact, if you try to do this, tsc will complain with an error like:
```
Type 'null' is not assignable to type 'string | undefined'
```
This alone seems like enough evidence that optional properties and `| undefined` typed properties are equivalent, or at least this tsc error seems to indicate as much

This time it seems that there is no difference between optional and `| undefined` typed class properties, so I would suggest declaring the property optional for the sake of readability.

## Function Parameters
Ok, so let's apply what we learned about class properties to function parameters. Suppose we have the following code:
```javascript
class MyClass {
  someMethod(param?: string) {
    if(param) {
      console.log(`param: ${param}`);
    }

    console.log('It works!');
  }
}

let foo = new MyClass();
foo.someMethod(null);
```
Notice how, once again, we are trying to use `null` when the code expects an optional. When we try to transpile this, we get a similar error from tsc saying that we can't assign `null` to a parameter of type `string | undefined`. Great! That means that optional parameters are just implicitly typed with an additional `| undefined`. If that's true, I say just declare it optional and make your readers happy.

One last thing to note here is there might be some value in declaring a function parameter `| undefined` typing to indicate to the caller that they _must_ supply this parameter, even if the value is `undefined`. I like that answer. It is subtle, and allows me to be really expressive with my code. But... is it correct?

For example:
```javascript
class MyClass {
  methodWithOptionalParam(param?: string) {
    console.log('It works!');
  }

  methodWithOrUndefinedParam(param: string | undefined) {
    console.log(`I made it!`);
  }
}

let foo = new MyClass();

// these two calls work
foo.methodWithOptionalParam();
foo.methodWithOptionalParam(undefined);

// this one also works - not shocking
foo.methodWithOrUndefinedParam(undefined);

// this call does not work!
foo.methodWithOrUndefinedParam();
```
The one and only tsc error we get with this code is on the very last call. The error claims `An argument for 'param' was not provided.` which means adding `| undefined` to function parameter typing _is_ meaningful in that it requires the caller to supply some parameter, even if that value is `undefined`.

But... why would you ever do that? I'm really not sure. Perhaps, I thought, this distinction is useful when specifying multiple optional parameters of the same type, like `myFunction(firstName?: string, lastName?: string)`. What happens when we call `myFunction('Kennedy')`? As it turns out, in this case 'Kennedy' is passed as the `firstName` parameter. That sort of implies that optional arguments are filled in from the left.

But what if the optional arguments have differing types? Surely TypeScript will use type inference to figure out what I mean, right? For example `myFunction(name?: string, age?: number)` what happens when we call `myFunction(10)`? It should use the typing to figure out that the first optional argument was not supplied and to assign the `10` to the `age` argument. Sadly, this is not true. Attempting to do this results in the following error from tsc:
```
Argument of type '10' is not assignable to parameter of type 'string | undefined'.
```
Even allowing for the fact that '10' isn't really a type, this error message makes me sad. In fact, the only way I can find to call this function with an unspecified first parameter is to call `myFunction(undefined, 10)` meaning, at least in these two cases of multiple optional params, I _already_ must supply the parameter as `undefined` without resorting to using `| undefined` typing on the parameter itself.

At this point I'm still at a loss as to why you might want to declare a function parameter as "you must supply this parameter, even if it is undefined" by using `| undefined` typing. Sure, there is a difference. I just don't know when I would choose to assert that difference on my callers.
