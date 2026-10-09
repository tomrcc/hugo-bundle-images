---
title: 'V3: static with placeholder, relative off'
date: "2026-10-01T09:00:00Z"
control: /images/control.jpg
cover: /cover.jpg
---
**Tests:** whether `static` fills in `[full_slug]`, with root-style values (`/cover.jpg`). Compare with V5, which writes the folder out in full.

**Result:**

- Load: broken. `src` is `/cover.jpg`, which isn't a URL on the site.
- Choose: the image browser opens under a literal `content/blog/[full_slug]` folder. You can't choose an image.
- Upload: saves to this folder but stores the full repo path (`/content/blog/v3/<file>`), which Hugo can't find. It previews straight away.
