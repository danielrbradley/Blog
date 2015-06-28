@{
    Layout = "post";
    Title = "Experiences of C# Async in a Web Application";
    Date = "2013-09-23T15:59:00";
    Tags = "csharp async";
    Description = "I’ve recently just done my first project embracing the new async features of C# 5.0 and thought I’d share some of my experiences using it out in the real world.";
}

## Why Use Async?
I’ve recently just done my first project embracing the new async features of C# 5.0 and thought I’d share some of my experiences using it out in the real world.

The project I was building was a web application which uses RavenDB for most persistence tasks. which has full support for running all operations that run network commands asynchronously. This then leads to the ability to run a full, end-to-end asynchronous stack, meaning that threads can be much better utilised, as a method that is waiting on a slow operations such as communicating across a network, can release its thread while waiting for the operation to complete.

## Writing Async Services & Interfaces
The approach I normally take when creating complex web applications is to use the concepts of services to decouple the implementation of the storage and business rules from the web front end. This works great for designing simple integration tests to confirm that a service method is working as expected. A normal service interface might look something like this:

    [lang=cs]
    public interface IService
    {
        SearchResult Search(Query query);
    }

To make this interface compatible with the async pattern, you simply wrap the return value of the action in a Task<>

    [lang=cs]
    public interface IService
    {
        Task<SearchResult> Search(Query query);
    }

If a service method does not have a return type (returns void), then simply return a plain Task.

    [lang=cs]
    public interface IService
    {
        Task Delete(string id);
    }

Async methods in this scenario should never return void, and should be considered an anti-pattern to avoid. More discussion is available in this MSDN article on Async Best Practices.

### Exceptions
One aspect of service design which is affected by the move to an asynchronous stack is how you model exceptional results that can be handled by the web application.

Typically, with synchronous services, custom exception types can be a neat way of representing unsuccessful results, which can optionally be caught at the web project level to be displayed nicely. However, in an asynchronous world, exceptions pose more challenges for handling due to possibly being wrapped in an aggregate exception.

Because of this, I took the approach of using polymorphic result types to represent all possible types of result (apart from system failure). This works much like a discriminated union in F# the return type is simply a finite list of possible outcomes.

    [lang=cs]
    public abstract class CreateResult
    {
        private CreateResult()
        {
        }

        public sealed class Success : CreateResult
        {
            public int Id { get; set; }
        }

        public sealed class NameNotUnique : CreateResult
        {
        }
    }

Using this kind of pattern, you can avoid using exceptions for any results that may want to be handled further down the stack.

## Testing (with NUnit)
Writing unit and integration tests around asynchronous methods was surprisingly straight forward with only a few minor problems (which may be fixed in future versions).

### Setup methods
The first problem I ran into with NUnit was the lack of support for writing an asynchronous setup method. NUnit currently requires the setup method to have a void return type and, as mentioned earlier, using a void return type with asynchronous methods is not fun. It means that the setup might not have finished setting everything up at the point when it starts running the test, resulting in inconsistent test environments.

The first solution to this is to not mark the setup method as "async" and run all asynchronous code synchronously using Task.WaitAll(), or some similar method.

The second solution is to create an asynchronous setup method, but rather than adding the NUnit setup attribute, call it directly on the first line of each test.

Neither of these methods is ideal, though I'd tend to prefer the second method as this allows for the setup method to be parameterised.

### Expected exceptions
The built-in NUnit asserts for expected exceptions do not support also checking inside of aggregate exception, and therefore I built my own assert helper class:

<script src="https://gist.github.com/danielrbradley/6671613.js"></script>
