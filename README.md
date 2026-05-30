# decouple

**Strict separation of configuration from code.**

`decouple` helps you organize your settings so that you can change parameters
without having to redeploy your app. It cleanly separates configuration values
(secrets, hostnames, feature flags, toggles) from your source code, so the same
codebase runs unchanged across local development, staging, and production —
only the environment changes.

> This repository is a fork of [HBNetwork/python-decouple](https://github.com/HBNetwork/python-decouple).
> See [Attribution](#attribution) below.

---

## Why decouple?

Hard-coding configuration in your source is a liability: secrets leak into
version control, environment-specific values force code edits, and every deploy
becomes risky. `decouple` follows the [Twelve-Factor App](https://12factor.net/config)
principle of storing config in the environment, while still giving you a
convenient fallback to local config files during development.

It lets you:

- Read settings from **environment variables** first, then fall back to a config file.
- Keep secrets **out of source control** (commit a `.env.example`, not your `.env`).
- **Cast** string values into the types your code actually needs (`int`, `bool`, lists, etc.).
- Provide sensible **defaults** so missing optional settings don't crash your app.

---

## Installation

```bash
pip install python-decouple
```

To install from this fork directly:

```bash
pip install git+https://github.com/tritathadore/decouple.git
```

---

## Quick start

### 1. Create a config file

`decouple` supports both `.env` and `.ini` formats. The simplest option is a
`.env` file in your repository root:

```ini
# .env
DEBUG=True
SECRET_KEY=your-secret-key-here
DATABASE_URL=postgres://user:password@localhost:5432/mydb
EMAIL_PORT=587
ALLOWED_HOSTS=.localhost,.example.com
```

Alternatively, a `settings.ini`:

```ini
# settings.ini
[settings]
DEBUG=True
SECRET_KEY=your-secret-key-here
DATABASE_URL=postgres://user:password@localhost:5432/mydb
EMAIL_PORT=587
```

> **Tip:** Add `.env` to your `.gitignore` and commit a `.env.example` with
> dummy values so collaborators know which variables to set.

### 2. Read your settings

```python
from decouple import config

SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
EMAIL_PORT = config('EMAIL_PORT', default=25, cast=int)
```

The `config` object is a lazy, auto-detecting factory: it searches up your module
path for a `settings.ini` or `.env` file and picks the right reader automatically.

---

## Casting values

Everything returned from the environment or a config file is a **string** by
default. Use `cast` to coerce values into the type you need.

### Booleans

```python
DEBUG = config('DEBUG', default=False, cast=bool)
```

Recognized truthy strings: `1`, `true`, `yes`, `y`, `on`, `t`.
Recognized falsy strings: `0`, `false`, `no`, `n`, `off`, `f`, `""`.

### Integers and floats

```python
EMAIL_PORT = config('EMAIL_PORT', default=587, cast=int)
TIMEOUT = config('TIMEOUT', default=30.0, cast=float)
```

### Lists with `Csv`

```python
from decouple import config, Csv

ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv())
# 'localhost,127.0.0.1' -> ['localhost', '127.0.0.1']

NUMBERS = config('NUMBERS', cast=Csv(int))
# '1,2,3' -> [1, 2, 3]
```

### Custom casting

`cast` accepts any callable, so you can pass your own function:

```python
SECURE_PROXY_SSL_HEADER = config(
    'SECURE_PROXY_SSL_HEADER',
    default='HTTP_X_FORWARDED_PROTO,https',
    cast=Csv(post_process=tuple),
)
```

---

## Defaults and required values

```python
# Optional setting with a fallback
CACHE_TIMEOUT = config('CACHE_TIMEOUT', default=300, cast=int)

# Required setting — raises UndefinedValueError if missing and no default given
SECRET_KEY = config('SECRET_KEY')
```

If a variable is not found in the environment **or** any config file, and no
`default` is provided, `decouple` raises an error early — failing fast instead
of silently running with bad config.

---

## Choosing the config source explicitly

By default, `config` auto-detects the file. To point at a specific source:

```python
from decouple import Config, RepositoryEnv

config = Config(RepositoryEnv('/path/to/.env'))
```

```python
from decouple import Config, RepositoryIni

config = Config(RepositoryIni('/path/to/settings.ini'))
```

---

## Precedence

`decouple` resolves values in the following order, stopping at the first hit:

1. **Environment variables** (`os.environ`)
2. **Repository** — your `.env` or `settings.ini` file
3. **The `default` argument** passed to `config()`

This Unix-style precedence means a value set in the actual environment always
wins over the config file, which is what you want in production.

---

## Django example

```python
# settings.py
from decouple import config, Csv

SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv())

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': config('DB_NAME'),
        'USER': config('DB_USER'),
        'PASSWORD': config('DB_PASSWORD'),
        'HOST': config('DB_HOST', default='localhost'),
        'PORT': config('DB_PORT', default=5432, cast=int),
    }
}
```

---

## Requirements

- Python 3.x
- No third-party runtime dependencies

---

## Contributing

Contributions are welcome. To set up a development environment:

```bash
git clone https://github.com/tritathadore/decouple.git
cd decouple
python -m venv venv
source venv/bin/activate
pip install -e .
```

Please open an issue to discuss substantial changes before submitting a pull request.

---

## Attribution

This project is a fork of [**python-decouple**](https://github.com/HBNetwork/python-decouple)
by Henrique Bastos and contributors. All credit for the original design and
implementation belongs to the upstream authors. This fork is maintained
separately and any modifications are documented in the commit history.

---

## License

`python-decouple` is released under the MIT License. This fork retains the
original license. See the [`LICENSE`](LICENSE) file for details.
