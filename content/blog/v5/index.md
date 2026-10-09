---
title: 'V5: static is this post''s folder, relative off'
date: "2026-10-01T09:00:00Z"
control: /images/control.jpg
cover: /cover.jpg
---
**Tests:** whether `static` pointing at the right folder helps root-style values. Same as V3, with the folder written out in full instead of a placeholder.

**Result:**

- Load: broken. `src` is `/cover.jpg`.
- Choose: works, stores `/other.jpg`, and previews straight away. It also works from the blog list.
- Upload: saves to this folder, stores `/<file>`, and previews straight away.

This is what V3 would do if `static` filled in `[full_slug]`.
