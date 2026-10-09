---
title: 'V11: static is content/, relative on'
date: "2026-10-01T09:00:00Z"
control: /images/control.jpg
cover: cover.jpg
---
**Tests:** V10 with relative paths on.

**Result:**

- Load: shows.
- Choose: the image browser opens in an empty folder (`content/content/blog/v11`). Going back to `blog > v11` works, and stores `other.jpg`.
- Upload: saves to this folder but stores `../../../blog/v11/<file>`, which is broken.
