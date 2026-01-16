# Testing BroTab with UV

This guide describes how to install and test the Manifest v3 version of `brotab` using the `uv` package manager on your local machine.

## Prerequisites

-   **Python 3.10+**
-   **`uv`**: Ensure you have `uv` installed.
-   **Google Chrome** or **Chromium** (or Brave).

## Installation

1.  **Clone the repository** (if you haven't already):
    ```bash
    git clone https://github.com/balta2ar/brotab.git
    cd brotab
    ```

2.  **Create a virtual environment and install dependencies**:
    ```bash
    uv venv
    uv pip install -e .
    ```

3.  **Activate the virtual environment**:
    ```bash
    # On macOS/Linux
    source .venv/bin/activate

    # On Windows
    # .venv\Scripts\activate
    ```

4.  **Install the Native Messaging Host**:
    Run the `bt install` command. This will generate the necessary JSON manifest files to allow the browser to communicate with the `brotab` backend.
    ```bash
    bt install
    ```

## Verification

### 1. Check the Native Messaging Manifest

Verify that the manifest file was created and points to the correct executable in your virtual environment.

**Linux/macOS:**
```bash
cat ~/.config/google-chrome/NativeMessagingHosts/brotab_mediator.json
```
(Adjust the path if you use Chromium or Brave, e.g., `~/.config/chromium/...`)

**Windows:**
Check the registry key or file at the location outputted by the `bt install` command.

**Important:** Look at the `"path"` field in the JSON output. It should point to the `bt_mediator` executable inside your `.venv` directory (e.g., `/path/to/brotab/.venv/bin/bt_mediator`).

### 2. Load the Extension in Chrome

1.  Open Chrome and navigate to `chrome://extensions`.
2.  Enable **Developer mode** (toggle in the top right).
3.  Click **Load unpacked**.
4.  Select the `brotab/extension/chrome` directory from this repository.

### 3. Verify Extension ID

The `brotab_mediator.json` file lists allowed extension IDs. Ensure the ID of the extension you just loaded matches one of the allowed origins in the manifest.

1.  In `chrome://extensions`, look at the ID of the "BroTab" extension.
2.  Check `brotab/mediator/chromium_mediator.json` (or the installed file checked in step 1).
3.  The ID should be one of:
    -   `mhpeahbikehnfkfnmopaigggliclhmnc` (Official Web Store ID)
    -   `knldjmfmopnpolahpmmgbagdohdnhkik` (Local Development ID specified in `manifest.json` "key")

If you are loading from source with the provided `key` in `manifest.json`, the ID should be `knldjmfmopnpolahpmmgbagdohdnhkik`, which is already allowed in the mediator manifest.

### 4. Test the Connection

1.  Restart Chrome to ensure it picks up the new Native Messaging Host.
2.  Open a few tabs in Chrome.
3.  From your terminal (with the virtual environment activated), run:
    ```bash
    bt clients
    ```
    You should see an entry for Chrome (e.g., `a.    google-chrome`).

4.  List the tabs:
    ```bash
    bt list
    ```
    You should see a list of your open tabs.

5.  Test closing a tab (optional):
    Find a tab ID from `bt list` (e.g., `a.123.456`) and run:
    ```bash
    bt close a.123.456
    ```

## Troubleshooting

-   **"No clients found"**:
    -   Make sure you restarted Chrome after `bt install`.
    -   Check if the path in `brotab_mediator.json` is absolute and points to the correct executable.
    -   Ensure the file permissions on `bt_mediator` allow execution (`chmod +x .venv/bin/bt_mediator`).
    -   Check Chrome's stdout/stderr if possible, or use the "background page" (service worker) inspector in `chrome://extensions` to look for connection errors in the console.

-   **Wrong Extension ID**:
    -   If your loaded extension has a different ID than the ones in `allowed_origins`, add your ID to `~/.config/google-chrome/NativeMessagingHosts/brotab_mediator.json` manually and restart Chrome.
