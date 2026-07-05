That's an even better feature. Your specification could be:

---

## Project: Python Function Signature Expander

### Objective

Build a static Python developer tool that automatically expands function calls by inserting all keyword arguments from the function signature.

### Behavior

When the user types:

```python
self.s3 = boto3.client(
    "s3",
)
```

the tool expands it to:

```python
self.s3 = boto3.client(
    "s3",
    # region_name="",
    # api_version="",
    # use_ssl=True,
    # verify="",
    # endpoint_url="",
    aws_access_key_id="",
    aws_secret_access_key="",
    # aws_session_token="",
    # config="",
)
```

### Rules

1. Detect the function being called.
2. Resolve the callable using static analysis (no AI).
3. Extract its complete signature.
4. Determine which parameters are **required** and which are **optional**.
5. Insert **required parameters** as active keyword arguments.
6. Insert **optional parameters** as commented-out keyword arguments.
7. Preserve the function's default values for optional parameters.
8. Do not duplicate arguments that the user has already supplied.
9. Preserve indentation and formatting.
10. Work for:

    * Standard library functions
    * Third-party libraries (e.g., `boto3`, `requests`, `pandas`)
    * User-defined functions

### Example

#### Input

```python
requests.get(
    url,
)
```

#### Output

```python
requests.get(
    url,
    # params=None,
    # headers=None,
    # cookies=None,
    # auth=None,
    # timeout=None,
    # allow_redirects=True,
)
```

### Technical Requirements

* Use `inspect.signature()` to obtain parameter information.
* Use `jedi` for symbol resolution.
* Use `libcst` (or `parso`) to parse and rewrite the source code while preserving formatting.
* Integrate with VS Code or any LSP-compatible editor as a command or code action.
* No generative AI or online services; rely entirely on static analysis.

This approach keeps the required arguments immediately usable while making all optional arguments visible and easy to enable by simply uncommenting them.
