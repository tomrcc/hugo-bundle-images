---
title: 'V2: static with placeholder, relative on'
date: "2026-10-01T09:00:00Z"
control: /images/control.jpg
cover: cover.jpg
---
**Tests:** whether `static` fills in `[full_slug]`.

**Result:**

- Load: shows.
- Choose: the image browser opens in `content/blog/[full_slug]/content/blog/v2`, which is empty. You can't choose an image.
- Upload: saves to this folder and stores a bare filename. It shows after the next build.
