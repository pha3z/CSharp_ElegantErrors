# C# Elegant Errors
Elegant Errors lets you write cleaner, more readable code by eliminating reliance on Exception Propagation

It will be most useful to you if:
- You write a lot of code that depends on API calls
- You write a lot of code that depends on methods that will fail for predictable reasons (such as unreachable network endpoint)
- You want your code to read in the same order it evaluates it.
- You want fewer bugs
- You want to be more explicit about where real program failures may occur

Stop writing code like this:

```csharp
public SomeType GetIt()
{
  try{
    var data = MakeRequest();
    //.. code here
    //.. and more code here
    //.. and more code still
    //.. you know what I'm talking about
    return Transform(data);
  }
  catch(Exception ex)
  {
    throw new CustomException("Could not GetIt.", ex)
  }

}
```

Start writing code like this:
```csharp
public R<SomeType> GetIt()
{
  var r = MakeRequest();
    if(r.isErr)
      return R<SomeType>.Err("Could not GetIt!", r);
    
  //.. code here
  //.. and more code here
  //.. and more code still
  //.. you know what I'm talking about
  return R.Ok(Transform(data));
}
```

## Why? Because Exceptions Should be Exceptional
Over-reliance on Exceptions makes ugly, brittle code that can even come with performance penalties. Here are some guidelines for using exceptions:

#### Do NOT use Exceptions for Normal Problem Domain Logic

For instance, when making API requests to a network, sometimes the network or server becomes unavailable (commonly in fact). This is a normal occurrence that you, the developer, should expect and plan for. When a network error occurs in an API request you've made, you may need to unwind up the stack several times to get back to an original method that will branch based on the failure. Depending on an exception to propagate up the stack for this scenario leaves ambiguity about everything that happens in between. Additionally, there is a strong tendency to get lazy and simply let try/catch blocks handle every single error case. This makes the code even harder to reason about.

#### Exceptions are for Real Program Failures or Mistakes, not Conditional Branching

If an exception occurs, it should really truly be something unexpected. Its a bug or a failure of the program to perform as expected. If you reserve Exceptions only for these situations, your code architecture will become clearer for two reasons:
Firstly, where you DO use try/catch blocks, you will know that you the code you've written there is meant to handle real full-blown program errors that you didn't account for. This lets you zero-in quickly on the places in your code where you are handling actual failures.
Secondly, you can bullet proof your own logic by throwing Exceptions for Unreachable Code or bizarre things that should never happen (such as a failed test case). The use of the "throw new Exception" pattern in these places will also make it crystal clear that these are real failures that your program didn't account for or a logic error you created.

#### Exceptions are SLOW

Throwing and catching even one single exception costs thousands of wasted processor cycles. They were never meant to occur frequently. This can really bite you in some situations.
For example: Imagine you write a web server that handles tens of thousands of requests, and this web server depends on calls to another API. If that other API fails and you start triggering thousands of exceptions, you might find that the API failure results in your server crashing to its knees and being unable to even respond to all of the requests. It will look like your own server is failing instead of handling the outage gracefully.

#### Exceptions are Hard to Follow

Developers who have written middleware or code that depends on a lot of calls to other APIs will be most familiar with this problem. A large portion of your code will involve methods which invoke some other method that might fail.  If the failure happens, you will probably want to bail immediately while returning some kind of response further up the stack. Only when things work successfully will you want to continue forward evaluating more code. Therefore, the code should follow that pattern to read like a book. Fail conditions should come first, followed by success conditions. Exceptions reverse this by assuming success and adding failure handling at the very end of a long indented block. Its NOT readable. If you think it is, you've simply gotten accustomed to a bad pattern. You need to break the habit and use a better one. Elegant Errors makes that possible.

## How To Use Elegant Errors

Include the ReturnResult.cs file in your project. It adds these classes:
- R
- R&lt;T&gt;

I named them the single letter R, because I didn't want to see "Result" or "Result&lt;T&gt;" everywhere in my code, but you can rename the classes to whatever you want.

Whenever you define a method that might normally fail due to some kind of undesirably situation, instead of throwing an exception, you will handle the exception internally. Then you will return R.Err().  You could also return R.Err() for any failure reason. It doesn't have to be due to something that would have caused an exception.

#### Write your method like this

```csharp
public R<YourReturnType> YourMethod(/*params*/)
{
  //Write your code here.
  //If an exception occurs, don't leave it unhandled to pass back up the stack (Unless its a real legitimate program error)
  //Instead, return an R object with an error
  
  //Do Something.
  //If it failed:
  return R<YourReturnType>.Err("Could not do something. There as a problem.");
  
  //Or maybe you do something that could cause an exception:
  try
  {
    //Do some code that may fail
  }
  catch(Exception ex)
  {
    //Notice that you can pass the exception to Err() method
    //The exception will be available to the consumer if consumer wants to examine it.
    return R<YourReturnType>.Err("Could not do it this way either! There was a problem!", ex);
  }
  
  //else you'll have some code that produces a return object (or primitive)
  var yourNewObject = new YourReturnType();
  return R.Ok(yourNewObject);
}
```
  
#### Calling Your Custom Method
Consumers who call YourMethod() will see that it returns an R of some T. This makes it self-documenting, because it is obvious that it has possible fail conditions that must be handled.

You can call your method like this:
```CSharp

var r = YourMethod(/*params*/);
if(r.isErr)
  //Handle the error here.  You could even return a new type of error to a caller.
  
//Else its a success.  Write your code here.
```

#### Additional Functions

There are some handy properties and methods to make it very easy to work with error messages. You can display them in a UI or make other logic based on them.

```CSharp

var r = YourMethod(/*params*/);

r.err //the error message
r.original_exception //the original exception that caused the error
r.hasMultipleErrors //if you create a new R and pass another R into it, this will be true
r.agg_errors //the err messages from any other Rs that were passed into the current R
r.AllErrors() //Concatenates the main err string with other aggregated error strings
r.JoinErrors(separator) //Joins all err strings together into a single string
```

## You Will Still need SOME Exception Handling

You will still need try/catch blocks in upper layers of your code to handle legitimate exceptions that weren't handled as R Errors. If you leverage these patterns correctly, you will have more explicit code. There will be a clearer distinction between code that is handling real unexpected program failures versus code that is handling fail conditions that can't always complete.

## DO NOT use the R Object Everywhere

There's no reason to use the R object in methods that aren't expected to fail. It would do nothing more than obvuscate your code. By using R return type only for methods that actually have regular failure conditions, you will show intent within your code. It will become easy to see which functions lead to branches with fail conditions that need to be handled.

Also, you might not want use the R object when a fail condition can be succinctly described. For instance, if your function always returns a positive non-zero integer on success, then you could simply return a 0 for failure cases... or a negative number. Do whatever makes your code light and easiest to understand at a quick glance.
