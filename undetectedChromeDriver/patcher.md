# patcher.py

This module will provide a class for patch changed and modified `chromedriver` for using as a webdriver for interacting with test script.

The purpose of this module is to make the webdriver less detectable by change it's default configurations. For instance:

## Differences between chrome webdriver and undetected webdriver

### 1. Navigator Object

#### 1.1 `navigator.webdriver` Property

- **Pure Selenium:** Returns `true`, signaling automation.
- **Undetected-Chromedriver:** Returns `false` or is hidden, bypassing basic bot detection.

- `__init__`()

This method will prepare class for download and creating new instance of chromedriver.
Some responsibilities like prepare path for storing new chrome driver instance, find and set chrome driver path and version and some actions like this.

> [!IMPORTANT]
> the Patch class has a __patch_exe__ method that change the actual chrome driver and set some changes to it. Changes are listed below

- `patch_exe`()

The `patch_exe` function is designed to modify the `chromedriver` executable file to make it less detectable by websites that use anti-bot measures. Here's a detailed breakdown of what it does:

#### 1. **Start Patching and Read File Content**

```python
start = time.perf_counter()
logger.info("patching driver executable %s" % self.executable_path)
with io.open(self.executable_path, "r+b") as fh:
    content = fh.read()
```

- Opens the `chromedriver` binary file (`self.executable_path`) in read and write binary mode (`"r+b"`).
- Reads the entire content of the file into the `content` variable.

#### 2. **Search for Injected Code Block**

```python
match_injected_codeblock = re.search(rb"\{window\.cdc.*?;\}", content)
```

- Uses a regular expression to find a specific code pattern in the binary data.
- The pattern `rb"\{window\.cdc.*?;\}"` is a **regex** that:
  - Looks for `{window.cdc...;}` in the binary data.
  - The `cdc` part refers to a well-known internal variable `window.cdc_` that `chromedriver` uses.
  - This variable can be a red flag for anti-bot scripts on websites.

#### 3. **Modify the Code Block if Found**

```python
if match_injected_codeblock:
    target_bytes = match_injected_codeblock[0]
    new_target_bytes = (
        b'{console.log("undetected chromedriver 1337!")}'.ljust(
            len(target_bytes), b" "
        )
    )
    new_content = content.replace(target_bytes, new_target_bytes)
```

- If the pattern is found:
  - `target_bytes` holds the original matched bytes.
  - `new_target_bytes` creates a new byte string with a fake script:

    ```js
    {console.log("undetected chromedriver 1337!")}
    ```

  - `ljust()` pads this string with spaces to match the length of the original `target_bytes`, ensuring the binary size remains unchanged.
  - `content.replace()` replaces the original code block with the new one.

#### 4. **Write Back to the File**

```python
    if new_content != content:
        fh.seek(0)
        fh.write(new_content)
```

- Seeks back to the start of the file (`fh.seek(0)`) and writes the modified content.
- If `new_content` is the same as `content`, it logs a warning.

#### 5. **Logging the Result**

```python
logger.info("patching took %.3f seconds" % (time.perf_counter() - start))
```

- Logs the time taken to complete the patching process.

#### **Purpose of the Patching**

The modification changes the internal variable or code that websites might check to detect automated `chromedriver` instances. By replacing it with harmless code, the bot attempts to avoid detection, allowing it to interact with websites that implement anti-bot mechanisms.
