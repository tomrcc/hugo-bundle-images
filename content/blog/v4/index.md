---
title: 'V4: static is this post''s folder, relative on'
date: "2026-10-01T09:00:00Z"
control: /images/control.jpg
cover: cover.jpg
---
**Tests:** whether `static` pointing at the right folder helps relative values. The folder is written out in full, with no placeholder.

**Result:**

- Load: shows.
- Choose: the image browser opens in an empty folder (`static` + `uploads`). Going back to `blog > v4` works, and stores `other.jpg`.
- Upload: saves to this folder but stores `../../../<file>`, which is broken.
