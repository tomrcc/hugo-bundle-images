---
title: 'V10: static is content/, relative off'
date: "2026-10-01T09:00:00Z"
control: /images/control.jpg
cover: /blog/v10/cover.jpg
---
**Tests:** storing the image's site URL (`/blog/v10/cover.jpg`). `static: content` makes the root-style value match the URL, with no placeholder. The templates remove the post's URL from the value before `.Resources.Get`.

**Result:**

- Load: shows, on this page and on the blog list.
- Choose: works, stores `/blog/v10/other.jpg`, and previews straight away. It also works from the blog list.
- Upload: saves to this folder, stores `/blog/v10/<file>`, and previews straight away. From the blog list it saves to `content/blog/blog/`: `[full_slug]` is filled from the list page.
