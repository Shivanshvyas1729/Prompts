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


In a code editor, there are a few good UI patterns. The cleanest one is **not** to insert method lists as comments into the source code, because they become stale and clutter the file.

Here are better approaches:

### Option 1: Hover Panel (Recommended)

When you hover over:

```python
self.s3
```

a popup appears:

```text
S3 Client

Common methods:
• upload_file()         Upload a local file
• upload_fileobj()      Upload a file-like object
• download_file()       Download to a local file
• get_object()          Read an object
• put_object()          Upload bytes
• list_objects_v2()     List objects

[Show All Methods]
```

Clicking **Show All Methods** opens the full list.

---

### Option 2: Completion List

When you type:

```python
self.s3.
```

the autocomplete dropdown becomes richer:

```text
📤 upload_file()        Upload a local file
📤 upload_fileobj()     Upload a file-like object
📥 download_file()      Download to a file
📄 get_object()         Read object
📄 put_object()         Upload object
📋 list_objects_v2()    List objects
```

Each item has a one-line description.

---

### Option 3: Side Panel

A dedicated explorer panel:

```text
S3 Client

Methods
├── upload_file()
├── upload_fileobj()
├── download_file()
├── get_object()
├── put_object()
└── ...

Properties
├── meta
├── exceptions
└── ...
```

Clicking a method inserts its call.

---

### Option 4: Smart Expansion (My Favorite)

When you choose:

```python
self.s3.upload_fileobj
```

the editor inserts:

```python
self.s3.upload_fileobj(
    Fileobj=,
    Bucket=,
    Key=,
    # ExtraArgs=None,
    # Callback=None,
    # Config=None,
)
```

Then, pressing a shortcut like **Ctrl+Space** on `upload_fileobj` shows:

```text
upload_fileobj()

Uploads a file-like object to S3.

Required
✓ Fileobj
✓ Bucket
✓ Key

Optional
• ExtraArgs
• Callback
• Config

Related
upload_file()
download_fileobj()
put_object()
```

---

### "Show All Methods"

This would simply be an action in the autocomplete or hover UI. For example:

```text
upload_file()
upload_fileobj()
download_file()
────────────────────
▶ Show All Methods…
```

Selecting it opens a searchable dialog:

```text
Search methods...

upload_file()            Upload a local file
upload_fileobj()         Upload a file-like object
download_file()          Download a local file
download_fileobj()       Download to a file object
copy()                   Copy an object
copy_object()            Low-level copy
delete_object()          Delete an object
generate_presigned_url() Create temporary URL
...
```

This is similar to the "Command Palette" or "Go to Symbol" interfaces developers are already familiar with.

---

If you're building this as a VS Code extension, I would avoid generating comments in the user's code. Instead, use **autocomplete items, hover popups, and code actions**. Those are the standard editor mechanisms, keep source files clean, and provide a much better user experience.


Yes. I think a **side panel** is a stronger design than putting comments into the code. It keeps the editor clean while giving much richer information.

A panel could look like this:

```text
──────────────────────────────────────────
📦 boto3.client("s3")
──────────────────────────────────────────

📄 upload_fileobj()

Uploads a file-like object to an S3 bucket.

──────────────────────────────────────────
Required Parameters
──────────────────────────────────────────
✓ Fileobj      Binary file-like object
✓ Bucket       Destination bucket
✓ Key          Object name

──────────────────────────────────────────
Optional Parameters
──────────────────────────────────────────
• ExtraArgs    Extra upload options
• Callback     Progress callback
• Config       Transfer configuration

[Insert Required]
[Insert All]

──────────────────────────────────────────
Related Methods
──────────────────────────────────────────
upload_file()        Upload a local file
download_fileobj()   Download to a file object
put_object()         Upload bytes directly

[Show More ▼]
```

When the user clicks **Show More**, the panel expands:

```text
──────────────────────────────────────────
All Methods (127)

upload_file()
upload_fileobj()
download_file()
download_fileobj()
copy()
copy_object()
delete_object()
get_object()
put_object()
list_objects_v2()
head_object()
generate_presigned_url()
...

🔍 Search methods...
```

Clicking a method updates the panel with its details.

### Even better ideas

You can organize methods into categories instead of one long list:

```text
📤 Upload (5)
    upload_file()
    upload_fileobj()
    put_object()

📥 Download (3)
    download_file()
    download_fileobj()
    get_object()

📂 Bucket (8)
    create_bucket()
    delete_bucket()
    list_buckets()

🔒 Security (4)
    generate_presigned_url()
    generate_presigned_post()

⚙️ Advanced (42)

▼ Show All
```

This makes navigation much easier for large SDKs like `boto3`, where clients often expose hundreds of methods.

You could also add **Quick Actions** at the top:

* **Insert Required Parameters**
* **Insert All Parameters**
* **Open Official Documentation**
* **Copy Example**
* **Show Usage Examples**
* **Go to Source** (if available)

This would make the extension feel more like a dedicated API explorer rather than just an autocomplete tool. It's especially useful for large libraries such as `boto3`, `pandas`, `torch`, and `tensorflow`, where developers often know *what* they want to do but not the exact method or its required arguments.



