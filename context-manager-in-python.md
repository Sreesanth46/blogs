---
title: Context manager in Python
date: 2024-05-20T19:23:00.000+00:00
---

# Context manager in Python

A context manager is a Python object that provides a clean way to manage resources, ensuring proper acquisition and release. By encapsulating the response formatting logic within a context manager, we can ensure consistency and maintainability throughout the project. It handles exceptions and sets appropriate status codes, enhancing consistency and robustness.

In Django projects, especially those involving larger teams, structuring response bodies for APIs can be a significant challenge. Inconsistent formats or lack of standardization can lead to confusion, potential issues, and decreased maintainability across the codebase. To address this problem, I leveraged the power of Python's context managers to ensure efficient and consistent response structuring.

Here's how I implemented a context manager to structure API response bodies in our Django project:


```python
class ResponseContext:
    def __init__(self):
        self.message = None
        self.status = None
        self.data = None

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type is not None:
            self.status = 400
            self.data = None
            self.message = str(exc_value)
            return True

    def get_response(self):
        response = {
            "message": self.message,
            "data": self.data
        }
        return JsonResponse(response, safe=True, status=self.status)


## Api views

@api_view(['GET'])
def get_user(request):
    with ResponseContext() as r: 
        r.message = "Success"
        r.data = User # Get user
        r.status = HTTP_200_OK 
    return r.get_response()
```

This `ResponseContext` class acts as a context manager, allowing us to wrap our response formatting code within a `with` statement. It handles exceptions and sets appropriate status codes, ensuring consistent and robust responses across our APIs.

### Bonus Tip

If you need to include error codes and messages as part of the response on errors, you could use a custom exception that raises an Exception with a status code and error code. You can then manage extracting the error code, message, and status code inside the `__exit__` block of the context manager.

```python
class CustomBaseException(Exception):

    def __init__(self, status_code, error_code):
        self.statusCode = str(status_code)
        self.errorCode = str(error_code)
        if error_code:
            self.message = getattr(message, error_code)
            
```

This `CustomBaseException` class raises a exception with `errorCode`, `statusCode` and `messsage`, now you can update the `__exit__` block as follows

```python
    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type is not None:
            if issubclass(exc_type, CustomBaseException):
                self.status = exc_value.statusCode
                self.data = None
                self.message = exc_value.message
                self.errorCode = exc_value.errorCode
            else:   
                self.status = 400
                self.data = None
                self.message = str(exc_value)
            return True
```

Now you just have to raise the exceptions in your views, response is managed by the context manager.

## Another use case

When working with databases in Django, it's important to properly manage the lifecycle of database connections and cursors. This can help prevent issues like resource leaks, performance problems, and unnecessary overhead.

By using the context manager, you can ensure that the cursor is properly closed and the database 
connection is returned to the connection pool, even in the event of an exception.
If you're working with a SQLite database, you can create a custom context manager that takes 
the database file as a parameter and manages the connection lifecycle.

Here's an example of a custom `SQLiteDBConnection` context manager:

```python
class SQLiteDBConnection:
    def __init__(self, db_file):
        self.db_file = db_file

    def __enter__(self):
        self.connection = sqlite3.connect(self.db_file)
        return self.connection.cursor()

    def __exit__(self, exc_type, exc_value, traceback):
        self.connection.close()

# Example usage

db_file = 'path/to/your/database.db'
with SQLiteDBConnection(db_file) as cursor:
    cursor.execute("SELECT * FROM my_table")
    results = cursor.fetchall()

# The connection is automatically closed at the end of the with block
```


fin.
