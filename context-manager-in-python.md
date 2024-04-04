# Context manager in Python

A context manager is a Python object that provides a clean way to manage resources, ensuring proper acquisition 
and release. By encapsulating the response formatting logic within a context manager, we can ensure consistency 
and maintainability throughout the project. It handles exceptions and sets appropriate status codes, enhancing 
consistency and robustness.

In Django projects, especially those involving larger teams, structuring response bodies for APIs can be a 
significant challenge. Inconsistent formats or lack of standardization can lead to confusion, potential issues, 
and decreased maintainability across the codebase. To address this problem, I leveraged the power of Python's 
context managers to ensure efficient and consistent response structuring.

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
    with ResponseContext() 
        r.message = "Success"
        r.data = User # Get user
        r.status = HTTP_200_OK 
    return r.get_response()
```

This `ResponseContext` class acts as a context manager, allowing us to wrap our response formatting code 
within a `with` statement. It handles exceptions and sets appropriate status codes, ensuring consistent 
and robust responses across our APIs.

### Bonus Tip

If you need to include error codes and messages as part of the response on errors, you could use a 
custom exception that raises an Exception with a status code and error code. You can then manage 
extracting the error code, message, and status code inside the `__exit__` block of the context manager.
