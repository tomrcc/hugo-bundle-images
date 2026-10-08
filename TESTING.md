# Bundle image regions test

Question: what input `paths` config lets an image region work on a page-bundle image? Both on the post's own page and on the blog list page.

On load, an image region always replaces `src` with `getPreviewUrl(<stored value>, <input config>)`. The image survives only if that URL points at a real file.

## Setup

Push to a new repo and connect it to a new CloudCannon site. `.cloudcannon/initial-site-settings.json` sets the build (`hugo mod vendor && hugo`, Hugo 0.166.0).

## Variants

Each post is in its own collection, so each has its own `cover` input. Every post shows a **Control** box (a `static/` image) and a **Cover** box (its bundle `cover.jpg`). The blog list page shows each post's cover again, bound with `@file[/content/blog/vN/index.md].cover`.

| Post | `static` | `uploads_use_relative_path` | Stored value |
| --- | --- | --- | --- |
| V1 | `""` | `true` | `cover.jpg` |
| V2 | `content/blog/[full_slug]/` | `true` | `cover.jpg` |
| V3 | `content/blog/[full_slug]/` | `false` | `/cover.jpg` |

All three use `uploads: content/blog/[full_slug]/`. V1 → V2 changes only `static`. V2 → V3 changes only `uploads_use_relative_path`, and the stored value changes with it to match what CloudCannon would write.

## Steps

On each post page and on the blog list page:

1. Open the page in the Visual Editor and don't touch anything. For each box: does the image show or is it broken? Inspect the `<img>` and copy its `src`.
2. For each cover: click it, choose `other.jpg` (the same image, flipped) from the post's folder, and close the panel. Does the flipped image show? What value does the sidebar `cover` input have? Don't save. Discard the change afterwards.

If a Control box is broken, something is wrong with that page. Stop and fix the page before reading its covers.

## Results

Post pages:

| Post | Control shows? | Cover shows? | Cover `src` | After `other.jpg`: shows? | Stored value |
| --- | --- | --- | --- | --- | --- |
| V1 | | | | | |
| V2 | | | | | |
| V3 | | | | | |

Blog list page (Control shows? ___ ):

| Post | Cover shows? | Cover `src` | After `other.jpg`: shows? | Stored value |
| --- | --- | --- | --- | --- |
| V1 | | | | |
| V2 | | | | |
| V3 | | | | |

What the results mean:

- V1 cover broken on the list page, but showing on its own page: a bundle image region only works where the page URL matches the bundle folder.
- V2 or V3 showing on the list page: that `paths` config fixes it everywhere.
- A `src` still containing `[full_slug]`: `static` doesn't expand placeholders.
- A stored value after choosing `other.jpg` that differs from the variant's format (`other.jpg` for V1/V2, `/other.jpg` for V3): CloudCannon writes a different form than this test assumes.
