---
title: 'V9: static left out, relative off'
date: "2026-10-01T09:00:00Z"
control: /images/control.jpg
cover: cover.jpg
---
**Tests:** what root-style values look like with no `static`.

**Result:**

- Load: shows (the starting value is a bare filename).
- Choose: stores `/content/blog/v9/other.jpg`, the full repo path. It previews, but Hugo can't find it.
- Upload: the same form of value. It previews straight away.
