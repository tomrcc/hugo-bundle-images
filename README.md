# Hugo bundle images test

How CloudCannon image editing behaves for Hugo page-bundle images (`content/blog/<post>/cover.jpg`, read with `.Resources.Get`). Raw results and steps are in [TESTING.md](TESTING.md).

## Reading the site

- **Each post is one variant.** `/blog/vN/` is in its own collection, so each has its own `cover` input `paths`. The variants are listed in `cloudcannon.config.yml` and [TESTING.md § Path combinations](TESTING.md#path-combinations-v1v11).
- **Post page:** two boxes, each an image region.
  - **Control:** a `static/` image. If it's broken, the page is broken; ignore its covers.
  - **Cover:** the post's bundle image.
  
  The text under each image shows the stored value and the `src` Hugo built.
- **Blog list (`/blog/`):** every post's cover again, as an image region bound with `@file`. This tests editing from another page.
- **Home page:** V1's cover in a component region (`layouts/partials/bundle-cover.html`), with no image region inside. The text under it shows:
  - `src`: the URL the partial built.
  - `via`: `resource` (`.Resources.Get` found the file) or `computed` (post URL + filename).
  - `renderer`: `build` or `editor`.
- **What to record in the editor:**
  - **Load:** the `<img>` `src` before touching anything.
  - **Choose:** the folder CloudCannon's image browser opens in, plus the stored value and `src` after choosing.
  - **Upload:** the stored value and the folder shown in the save modal.

## Findings

- **Image regions use the raw stored value when the page loads.** The value only shows if it already works as a URL from the current page.
- **After a change, root-style values (`/…`) resolve through `static`** to a CloudCannon file URL, so a choose or an upload previews straight away. Relative values don't, so a new upload only shows after the next build.
- **`uploads_use_relative_path` changes how the value is written, not where the file goes.**
  - On: relative to the edited file (`cover.jpg`).
  - Off: relative to `static` (`/cover.jpg`).
- **`static` doesn't fill in placeholders.** `[full_slug]` stays literal, so the image browser opens in a folder that doesn't exist.
- **`uploads: ""` means the repo root.**
- **`.Resources.Get` accepts a leading slash.** `"/cover.jpg"` finds the bundle's `cover.jpg`.
- **The editor's Hugo can't see bundle files.** A component partial has to build the URL itself: the post's `.RelPermalink` plus the filename.

| Setup | Load | Choose | Upload preview | Other pages |
| --- | --- | --- | --- | --- |
| V1: `static: ""`, relative on (`cover.jpg`) | Own page only | ✅ | After rebuild | Component region only |
| V5: `static` = post folder hard-coded, relative off (`/cover.jpg`) | ❌ | ✅ | ✅ | ✅ choose (load ❌) |
| **V10: `static: content`, relative off (`/blog/v10/cover.jpg`)** | ✅ | ✅ | ✅ | to confirm |

**V10** stores the image's site URL. It needs no placeholder in `static`.

Its costs:
- The template strips the post's own URL from the value before `.Resources.Get` (see `layouts/page.html`).
- Values break if a post's URL stops matching its folder (permalinks, slug changes).
