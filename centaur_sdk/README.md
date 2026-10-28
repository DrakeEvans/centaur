# Centaur SDK

Lightweight toolkit for building Centaur-compatible tools.

## Install

```bash
# From git
pip install "centaur-sdk @ git+https://github.com/<owner>/<repo>.git#subdirectory=centaur_sdk"

# With HTTP backend support
pip install "centaur-sdk[http] @ git+https://github.com/<owner>/<repo>.git#subdirectory=centaur_sdk"
```

## Usage

### Secrets

```python
from centaur_sdk import secret

api_key = secret("MY_API_KEY")
```

By default, `secret()` uses `StubBackend`: it returns the key name as a
placeholder, and iron-proxy replaces it with the real credential on matching
outbound requests. When the key is set in the environment, the environment
value is returned instead.

To run a tool outside a sandbox against real credentials in your shell,
configure `EnvBackend` first. Don't use it in code that runs in a sandbox.

```python
from centaur_sdk.backends import EnvBackend, configure

configure(EnvBackend())
```
