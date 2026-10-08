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
| 1. Load | Yes | | | | — |
| 2. Choose `other.jpg` | Yes | `/blog/v1/other.jpg` | `computed` | `editor` | `other.jpg` |
| 3. Upload | No | `/blog/v1/send-it-ai.png` | `computed` | `editor` | `send-it-ai.png` (file written to `content/blog/v1/`) |

Step 3 note: the value and file location are correct. The preview breaks because `/blog/v1/send-it-ai.png` only exists in the built site after the next build. After save and rebuild it shows on both V1's page and the home page. An upload through the image region on V1's own page doesn't preview before a rebuild either, so the gap isn't specific to the component.

What the results mean:

- Step 2 shows the flipped image with `via computed`, and the stored value is `other.jpg`: a component region works off-page. The editor can't see bundle files, so the partial has to build the URL itself.
- `via resource` with `renderer editor`: the editor can see bundle files, so a plain `.Resources.Get` partial works.
- Stored value `v1/other.jpg`, or anything other than a bare filename: the panel saves relative to the page being edited, as the image region did. The post's own page would break after a save.
- Cover blank, or a `Failed to render Hugo component` error: the lookup by title failed in the editor's Hugo.

## Static path variant (V4)

Question: does a fresh bundle upload preview before a rebuild if `paths.static` points at the post's folder?

V2 and V3 couldn't answer this, because `static: content/blog/[full_slug]/` kept `[full_slug]` literal. V4 writes the folder out in full (`static: content/blog/v4/`) and is otherwise the same as V1. A hard-coded per-post path isn't usable on a real site; this only tests whether the mapping helps.

Steps:

1. **Control upload (baseline).** On V4's page, upload a new image to the Control box. Does it show straight away? Inspect the `<img>` and copy its `src`. Discard.
2. **V4 load.** Reload V4's page without touching anything. Does the cover show? Copy its `src`.
3. **V4 choose.** Click the cover. Which folder does the picker open in? Choose `other.jpg`. Does it show? Copy its `src` and the stored value. Discard.
4. **V4 upload.** Click the cover and upload a new image. Does it show straight away? Copy its `src` and the stored value, and note which folder the file was written to. Discard.

| Step | Shows? | `src` | Picker folder | Stored value |
| --- | --- | --- | --- | --- |
| 1. Control upload | | | — | |
| 2. V4 load | | | — | `cover.jpg` |
| 3. V4 choose `other.jpg` | | | | |
| 4. V4 upload | | | | |

What the results mean:

- Step 1 previews and step 4 previews: a correct `static` mapping lets unbuilt uploads preview. Only the placeholder support is missing, which strengthens UPSTREAM-DRAFTS #20b.
- Step 1 previews and step 4 doesn't: `static` mapping doesn't help bundle files. Compare the two `src` values to see what CloudCannon does differently for `static/`.
- Step 1 doesn't preview: no upload previews before a rebuild, whatever the paths.
- Step 2 or 3 broken: a full-path `static` breaks existing images. Compare with V1, where both work.
