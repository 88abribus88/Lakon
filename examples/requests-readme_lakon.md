---
source: /home/abribus/workspace/lakon/examples/requests-readme.md
translated_at: 2026-05-06T08:58:12Z
skill: skills/translate
version: "1.0"
---

# Requests

Requests — simple + elegant HTTP library.

```python
>>> import requests
>>> r = requests.get('https://httpbin.org/basic-auth/user/pass', auth=('user', 'pass'))
>>> r.status_code
200
>>> r.headers['content-type']
'application/json; charset=utf8'
>>> r.encoding
'utf-8'
>>> r.text
'{"authenticated": true, ...'
>>> r.json()
{'authenticated': True, ...}
```

Requests sends HTTP/1.1 requests with no manual query string or form-encoding — use `json` method instead.

`300M downloads / week`; depended on by 4,000,000+ repositories per GitHub.

## Installing Requests + Supported Versions

Available on PyPI:

```console
$ python -m pip install requests
```

Supports Python 3.10+.

## Supported Features + Best Practices

Ready for robust, reliable HTTP applications.

- Keep-Alive + Connection Pooling
- International Domains + URLs
- Sessions with Cookie Persistence
- Browser-style TLS/SSL Verification
- Basic + Digest Authentication
- `dict`-like Cookies
- Automatic Content Decompression + Decoding
- Multi-part File Uploads
- SOCKS Proxy Support
- Connection Timeouts
- Streaming Downloads
- Automatic `.netrc` honoring
- Chunked HTTP Requests

## Cloning the Repository

May need `-c fetch.fsck.badTimezone=ignore` to avoid bad-commit-timestamp error (see [this issue](https://github.com/psf/requests/issues/2690)):

```shell
git clone -c fetch.fsck.badTimezone=ignore https://github.com/psf/requests.git
```

Apply globally:

```shell
git config --global fetch.fsck.badTimezone ignore
```
