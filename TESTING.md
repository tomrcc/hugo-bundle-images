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

## Component region variant

Question: can a component region show and live-update a bundle image off its own page, where an image region can't?

The home page shows V1's cover inside a component region (`bundle-cover` partial) bound to `@file[/content/blog/v1/index.md]`. There is no image region inside it. The partial looks up the post by title, then tries `.Resources.Get` and falls back to the post's URL plus the filename. The text under the image shows which path it took (`via resource` or `via computed`) and which renderer produced it (`build` or `editor`). Only V1 is tested, because V1 has the config the skill recommends.

Steps on the home page:

1. Open it in the Visual Editor and don't touch anything. Does the cover show? Note the `src`, `via` and `renderer` text.
2. Click the component and change `cover` to `other.jpg` in its panel, then close the panel. Does the flipped image show? Note the `src`, `via` and `renderer` text, and the stored value in the panel. Don't save. Discard the change afterwards.
3. Upload a new image to `cover` from this panel. Where does the upload go, and what value is stored? Discard afterwards.

| Step | Cover shows? | `src` | `via` | `renderer` | Stored value |
| --- | --- | --- | --- | --- | --- |
| 1. Load | | | | | — |
| 2. Choose `other.jpg` | | | | | |
| 3. Upload | | | | | |

What the results mean:

- Step 2 shows the flipped image with `via computed`, and the stored value is `other.jpg`: a component region works off-page. The editor can't see bundle files, so the partial has to build the URL itself.
- `via resource` with `renderer editor`: the editor can see bundle files, so a plain `.Resources.Get` partial works.
- Stored value `v1/other.jpg`, or anything other than a bare filename: the panel saves relative to the page being edited, as the image region did. The post's own page would break after a save.
- Cover blank, or a `Failed to render Hugo component` error: the lookup by title failed in the editor's Hugo.
