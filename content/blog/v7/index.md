---
title: 'V7: uploads and static left out, relative on'
date: "2026-10-01T09:00:00Z"
control: /images/control.jpg
cover: cover.jpg
---
**Tests:** the docs example, which sets only `uploads_use_relative_path`.

**Result:**

- Load: shows.
- Choose: works, stores `other.jpg`.
- Upload: saves to `/uploads/` and stores `../../../uploads/<file>`. A left-out `uploads` is CloudCannon's default `uploads/` folder.
