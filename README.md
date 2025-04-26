# ThreadsClient

A lightweight, **unofficial** Python wrapper around the Threads Graph API that lets you publish text, image, video and carousel posts—and full reply threads—in a single call.

> **Status** : Alpha • API surface **may change** without notice

---

## ✨ Features

| ✔ | Capability | Notes |
|---|-------------|-------|
| Post text, image or video | Single‑media posts | Automatically detects image vs video
| Carousels (≤ 10 items) | Mixed images & videos | Handles container creation + publish flow
| Reply chains | Any depth | Each reply can itself be single or carousel
| Token validation | `verify_access_token()` | Light ping to `/me` endpoint
| Rich status dict | Mirrors raw API IDs & timestamps | Useful for your own DB‑logging
| Time‑zone aware | JST timestamps (Asia/Tokyo) | Easy to swap if needed
| CL I helper | `python threads_client.py <token> <user> data.json` | For quick tests

---

## 🔧 Installation

```bash
# Clone the repo (private for now)
$ git clone https://github.com/YOUR_ORG/threads-client.git
$ cd threads-client

# Install into the current env
$ pip install -U .
```

or

```bash
# PyPI
pip install ThreadsPost
```

> The library is pure‑Python (3.8+) and has **only two runtime deps**: `requests` and `pytz`.

---

## 🚀 Quick‑start

```python
from threads_client import ThreadsClient

ACCESS_TOKEN = "<long‑lived user token>"
USER_ID      = "insane_nao"   # ←  without the leading "@"

client = ThreadsClient(access_token=ACCESS_TOKEN, user_id=USER_ID)

main_post = {
    "content": "📸 Trip highlights!",
    "media": ["https://example.com/photo1.png", "https://example.com/video1.mp4"],  # <= 10 items
}

replies = [
    {"content": "Behind the scenes", "media": []},
    {"content": "Full clip here", "media": ["https://example.com/video2.mp4"]},
]

status = client.post(main_post, replies)
print(status["postUrl"])  # → https://www.threads.net/@insane_nao/post/⋯
```

---

## 🛠 API Reference

### `ThreadsClient(access_token: str, user_id: str)`
Creates a client bound to a single user.

| Param | Type | Description |
|-------|------|-------------|
| `access_token` | `str` | Long‑lived user token obtained from Meta App Dashboard. |
| `user_id` | `str` | Threads username **without** the `@`. |

---

### `post(main_post: dict, reply_posts: list[dict] | None = None) -> dict`
Publishes a root post then optionally a chain of replies.

`main_post` / `reply_posts[i]` schema
```python
{
  "content": "Hello world",           # str – optional if `media` present
  "media": [                          # list[str] – ≤ 10
      "https://…/image.png",          # supports .jpg/.png
      "https://…/clip.mp4",           # or .mp4/.mov
  ]
}
```

Returns a **status dict** that includes every container/post ID, creation timestamps (JST) and the final permalink.

---

### `verify_access_token() -> bool`
Pings `/me` and returns **True** if the token is valid.

---

## ⚙️ Environment & Limits

* **Threads Graph API v1.0** endpoints are hard‑coded (`https://graph.threads.net/v1.0`).
* Carousels are capped at **10 media items** (`MAX_MEDIA_NUM`).
* Media processing timeout: **3 minutes** per item.
* Publish retries: single (60 attempts × 3 s) · carousel (300 × 5 s).

Adjust these constants in `threads_client.py` if your needs differ.

---

## ❗ Error Handling
All HTTP and logical failures raise **`ThreadsClientError`** with the raw API payload in `.args[0]`.

```python
from threads_client import ThreadsClient, ThreadsClientError
try:
    client.post(main_post)
except ThreadsClientError as exc:
    # `exc.args[0]` → JSON body from Threads API
    raise
```

---

## 📝 License

```
MIT License

Copyright (c) 2025 Nao Matsukami

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 🙏 Acknowledgements

This wrapper is merely syntactic sugar over Meta's official Threads Graph API. Kudos to the engineering teams who built the underlying infrastructure.

