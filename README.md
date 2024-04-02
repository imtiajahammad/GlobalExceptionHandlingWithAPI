## Global Exception Handling Implementaion with Asp.Net Core WebAPI

In context of webapi, Global Exception Handling involves catching exceptions that occur during the 
- processing of requests
- returning appropriate error responses to the clients
Instead of allowing exceptions to occur in the top level and exposing technical details to the end-users. Global Exception Handling can convert those exceptions into more informative and standaridized error messages in a format such as JSON or XML.
### Key Benefits of using Global Exception Handling
- **Consistent Error Responses**: It ensures that all exceptions are handled and translated into a consistent format, to make it easier for clients to understand and handle errors uniformly.
- **Imporoved User Experience**: Instead of desplaying cryptic error messages or stack traces, Global Exception Handling can present user-friendly messages that explain the error in a human-readable way.
- **Application Resilience**: By handling exceptions at a central level, the appplication can gracefully recover from errors and continue functioning without crashing
- **Logging and Monitoring**: It allows you to log exception details, helping you to monitor and diagnose application issues in production environments.
- **Security**: By not exposing low-level technical details of exceptions to users, you can prevent potential security vulnerabilities.


In ASP.NET Core, global exception handling is typically implemented using middleware. Middleware is a pipeline component that can inspect, modify, or terminate incoming requests and outgoing responses. We can create custom middleware to handle exceptions and register it in the application pipeline.

A common approach to implementing global exception handling involves-
- Creating a custom middleware that catches exceptions and generates appropriate error responses
- Registering the custom middleware in the ASP.NET Core application pipeline, ensuring it's invoked for all requests
- Handling various types of exceptions(application-specific exceptions, system-level exceptions) and returning appropriate status codes and messages

Global exception handling can also work in conjunction with logging frameworks, such as Serilog or NLog, to log exception details for better monitoring and debugging.

However, it's essential to use global exception handling judiciously. Some exceptions may indicate unrecoverable error or security vulnerabilities, and handling them globally might not be appropriate. In such cases, we may need to let those exceptions propagate up to the top-level error handler or handle them specifically in certain parts of your application.

Also to remember, while global exception handling can provide a safety net for unhandled exceptions it is also crucial to design our application in a way that minimizes the occurrence of exceptions and to thoroughly test your code to catch any potential issues early in the development process.

1. Open Terminal and Go to your preferred directory and make a folder for your project solution
    ```
    mkdir GlobalExceptionHandlingWithAPI
    cd GlobalExceptionHandlingWithAPI
    ```
2. Create a blank solution with name **GlobalExceptionHandlingWithAPI**
    ```
    dotnet new sln -n GlobalExceptionHandlingWithAPI
    ```
3. Create a webapi project in the solution
    ```
    dotnet new webapi -f net6.0 -n GlobalExceptionHandlingWithAPI
    ```
4. Add the project into the solution 
    ```
    dotnet sln add GlobalExceptionHandlingWithAPI/GlobalExceptionHandlingWithAPI.csproj
    ```
5. Create a gitignore file in the solution
    ```
    dotnet new gitignore
    ```
6. Create a readme.md file 
    ```
    touch README.md 
    ```
7. Now open the solution in visual studio code
    ```
    code .
    ```
8. Install the following packages
    ```
    dotnet add package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore -v 6.0.28
    dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson -v 6.0.28
    ```
9. Create a custom exception class. We can create a custom exception class to represent specific application-level exceptions. This can be useful for adding more context to the errors.
    ```
    dotnet new class -n CustomException
    ```
    ```
    public class CustomException : Exception
    {
    public CustomException(string message) : base(message)
    {
        
    }
    }
    ```
10. Create a middleware class to handle global exceptions. The middleware will catch unhandled exceptions and convert them into appropriate HTTP responses.
    ```
    dotnet new class -n GlobalExceptionMiddleware
    ```
    ```
    public class GlobalExceptionMiddleware
    {
        private readonly RequestDelegate _next;

        public GlobalExceptionMiddleware(RequestDelegate next)
        {
            _next = next;
        }
        public async Task InvokeAsync(HttpContext context)
        {
            try
            {
                await _next(context);
            }
            catch(Exception ex)
            {
                await HandleExceptionAsync(context, ex);
            }
        }

        private static Task HandleExceptionAsync(HttpContext context, Exception exception)
        {
            context.Response.ContentType = "application/json";
            context.Response.StatusCode = (int) HttpStatusCode.InternalServerError;

            var errorResponse = new
                {
                    message = "An error occurred while processing your request.",
                    details = exception.Message
                };

                var jsonErrorResponse = JsonConvert.SerializeObject(errorResponse);

                return context.Response.WriteAsync(jsonErrorResponse);
        }
    }
    ```
11. Register the global exception handling middleware in program file
    ```
    //Registering the global exception handling middleware 
    app.UseMiddleware<GlobalExceptionMiddleware>();
    ```


#### Reference: https://www.c-sharpcorner.com/article/global-exception-handling-in-asp-net-core-web-api/
