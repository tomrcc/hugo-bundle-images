---
title: 'V6: uploads and static empty, relative on'
date: "2026-10-01T09:00:00Z"
control: /images/control.jpg
cover: cover.jpg
---
**Tests:** whether an empty `uploads` means "this post's folder".

**Result:**

- Choose: works, stores `other.jpg`.
- Upload: saves to the repo root and stores `../../../<file>`. An empty `uploads` is the repo root.
