
# Simple.Result

A small and simple C# library for representing operation results in a consistent and type-safe way.  

The library provides two main types: `Result` (non-generic) and `Result<T>` (generic), backed by a `Status` enum describing possible operation outcomes.

## Features

- Clear representation of both successful and failed operations.
- Supports carrying messages and exceptions.
- Generic version (`Result<T>`) that holds a value when the operation is successful.
- Factory methods for easy creation of different result types (Success, Created, NotFound, Exception, etc.).

## Installation

`dotnet add package Simple.Result`

## API Overview

```csharp
// Non-generic
public class Result
{
    public Status Status { get; init; }
    public string? Message { get; init; }
    public Exception? Exception { get; init; }
    public bool IsSuccess { get; }

    public static Result OfSuccess(string? message = null) { ... }
    public static Result OfCreated(string? message = null) { ... }
    public static Result OfNotFound(Exception? exception, string? message = null) { ... }
    public static Result OfException(Exception exception, string? message = null) { ... }
    // ... and more factory methods
}

// Generic
public class Result<T> : Result
{
    public T? Value { get; init; }
    public Type Type => typeof(T);

    public static Result<T> OfSuccess(T value, string? message = null) { ... }
    public static Result<T> OfCreated(T? value = default, string? message = null) { ... }
    public new static Result<T> OfException(Exception exception, string? message = null) { ... }
    // ... and more 
}

// Available statuses
public enum Status
{
    Success = 0,
    FileStreamResult,
    NoOperation,
    Created,
    ClientError,
    ServerError,
    NotFound,
    Conflict,
    AccessDenied,
    ContentTooLarge,
    Exception
}
```

## Examples

### Using non-generic `Result`
```csharp
Result r = Result.OfSuccess("Saved successfully");
if (r.IsSuccess)
{
    Console.WriteLine(r.Message); // "Saved successfully"
}
```

### Using generic `Result<T>`
```csharp
Result<User> r = Result<User>.OfSuccess(new User { Id = 1, Name = "Anna" });
if (r.IsSuccess)
{
    Console.WriteLine(r.Value.Name); // "Anna"
}
else
{
    Console.WriteLine(r.Message);
}
```

### Returning from an API Controller (ASP.NET Core)
```csharp
[HttpGet("{id}")]
public ActionResult<UserDto> GetUser(int id)
{
    var result = _userService.GetUser(id); // Result<UserDto>

    if (result.IsSuccess)
    {
        return Ok(result.Value); 
    }

    return result.Status switch
    {
        Status.NotFound => NotFound(result.Message),
        Status.AccessDenied => Forbid(),
        Status.ClientError => BadRequest(result.Message),
        Status.ServerError => StatusCode(500, result.Message),
        _ => StatusCode(500, result.Message)
    };
}
```

## Design Rationale

- The `Exception?` property is included to preserve caught exceptions without losing context.
- `IsSuccess` is defined to include multiple “positive” statuses (`Success`, `Created`, `NoOperation`, `FileStreamResult`) for easier success checks.
- Factory methods (`OfSuccess`, `OfNotFound`, `OfException`, etc.) provide consistent and readable construction of result instances.

## Tips & Best Practices

- Use `Result<T>` when your operation should return a value; use the non-generic `Result` for status-only operations.
- Avoid using the `Exception` property for business logic; keep it for diagnostics and logging.
- Keep messages (`Message`) user-friendly if exposed externally; use logging for detailed technical errors.

## License

MIT — see the LICENCE file for details.
