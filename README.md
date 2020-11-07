# C# Elegant Errors
Elegant Errors lets you write cleaner, more readable code by eliminating reliance on Exception Propagation

Stop writing code like this:

```csharp
public SomeType GetIt()
{
  try{
    var data = MakeRequest();
    //... code here
    //.. and more code here
    var SomeObject = Transform(data);
    return someObject;
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
    
  //... code here
  //.. and more code here
  var SomeObject = Transform(data);
  return R.Ok(someObject);
}
```

Over-reliance on Exceptions makes ugly, brittle code that can even come with performance penalties. Here are some guidelines for using exceptions:

#### Do NOT use Exceptions for Normal Problem Domain Logic

For instance, when making API requests to a network, sometimes the network or server becomes unavailable (commonly in fact). This is an excepted and normal occurrence that you, the developer, should expect and plan for. When a network error occurs in an API request you've made, you may need to unwind up the stack several times to get back to an original method that will branch based on the failure. Depending on an exception to propagate up the stack for this scenario will make your code very ugly. Additionally, there is a strong tendency to get lazy and simply let try/catch blocks handle every single error case. This makes the code even harder to reason about.

#### Exceptions are for Real Program Failures or Mistakes, not Conditional Branching

If an exception occurs, it should really truly be something unexpected. Its a bug or a failure of the program to perform as expected. If you reserve Exceptions only for these situations, your code architecture will become clearer for two reasons:
Firstly, where you DO use try/catch blocks, you will know that you the code you've written there is meant to handle real full-blown program errors that you didn't account for. This lets you zero-in quickly on the places in your code where you are handling actual failures.
Secondly, you can bullet proof your own logic by throwing Exceptions for Unreachable Code or bizarre things that should never happen (such as a failed test case). The use of the "throw new Exception" pattern in these places will also make it crystal clear that these are real failures that your program didn't account for or a logic error you created.

#### Exceptions are SLOW

Throwing and catching even one single exception costs thousands of wasted processor cycles. They were never meant to occur frequently. This can really bite you in some situations.
For example: Imagine you write a web server that handles tens of thousands of requests, and this web server depends on calls to another API. If that other API fails and you start triggering thousands of exceptions, you might find that the API failure results in your server crashing to its knees and being unable to even respond to all of the requests. It will look like your own server is failing instead of handling the outage gracefully.

#### Exceptions are Hard to Follow

Developers who have written middleware or code that depends on a lot of calls to other APIs will be most familiar with this problem. A large portion of your code will involve methods which invoke some other method that might fail.  If the failure happens, you will probably want to bail immediately while returning some kind of response further up the stack. Only when things work successfully will you want to continue forward evaluating more code. Therefore, the code should follow that pattern to read like a book. Fail conditions should come first, followed by success conditions. Exceptions reverse this by assuming success and adding failure handling at the very end of a long indented block. Its NOT readable. If you think it is, you've simply gotten accustomed to a bad pattern. You need to break the habit and use a better one. Elegant Errors makes that possible.
