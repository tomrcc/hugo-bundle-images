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
| 1. Control upload | Yes, until a refresh without saving | `/images/screenshot-2026-06-05-at-12-37-42-pm.png` | — | |
| 2. V4 load | Yes | `cover.jpg` | — | `cover.jpg` |
| 3. V4 choose `other.jpg` | Yes | `other.jpg` | Shown as `content/blog/v4`, but empty; had to go back several levels to find `other.jpg` | |
| 4. V4 upload | No | `../../../pexels-enzo-elgalgo.jpg` | — | `../../../pexels-enzo-elgalgo.jpg`; file landed beside `cover.jpg` |

Step 4 note: the picker path is shown relative to `static`, so `content/blog/v4` there is probably `content/blog/v4/content/blog/v4/`. `../../../` is the relative path from that doubled folder back to the real one. Because the stored value is wrong, V4 can't answer the preview question.

What the results mean:

- Step 1 previews and step 4 previews: a correct `static` mapping lets unbuilt uploads preview. Only the placeholder support is missing, which strengthens UPSTREAM-DRAFTS #20b.
- Step 1 previews and step 4 doesn't: `static` mapping doesn't help bundle files. Compare the two `src` values to see what CloudCannon does differently for `static/`.
- Step 1 doesn't preview: no upload previews before a rebuild, whatever the paths.
- Step 2 or 3 broken: a full-path `static` breaks existing images. Compare with V1, where both work.

## Static path variant, relative off (V5)

Question: same as V4, with values stored the way the Control box stores them.

V5 is V4 with `uploads_use_relative_path: false` and `cover: /cover.jpg`. The Control input works the same way (`static: static`, relative off, `/images/…`). Hugo's `.Resources.Get` finds `/cover.jpg` in the bundle, so the build is unchanged.

Steps, on V5's page (`/blog/v5/`). Copy each `src` straight away, before any refresh:

5. **Load.** Reload without touching anything. Does the cover show? Copy its `src`.
6. **Choose.** Click the cover. Where does the picker open? Choose `other.jpg`. Does it show? Copy its `src` and the stored value. Discard.
7. **Upload.** Click the cover and upload a new image. Does it show straight away? Copy its `src` and the stored value, and note where the file went. Discard.

| Step | Shows? | `src` | Picker folder | Stored value |
| --- | --- | --- | --- | --- |
| 5. V5 load | No | `/cover.jpg` | — | `/cover.jpg` |
| 6. V5 choose `other.jpg` | Yes | `https://app.cloudcannon.com/api/v0/sites/<site>/files/content%2Fblog%2Fv5%2Fother.jpg?…` | The bundle folder (no path shown) | `/other.jpg` |
| 7. V5 upload | Yes, straight away | `https://app.cloudcannon.com/api/v0/editing_session_files/<id>/raw?…` | — | `/pexels-karolina-grabowska.jpg`; save modal path `/content/blog/v5/` |

Step 5–7 note: the image region calls `getPreviewUrl(value, inputConfig)` on load and on change (`editable-regions/nodes/editable-image.ts:136`). On load it returned the raw value. After a change it used `static` to resolve the value to the repo file (or the unsaved upload) through CloudCannon's API. So a correct `static` does make bundle edits and uploads preview, but the unchanged value on load still breaks.

What the results mean:

- Step 7 previews with a `/…` `src`: a correct `static` mapping lets unbuilt bundle uploads preview, as it does for Control. Only placeholder support in `static` is missing (UPSTREAM-DRAFTS #20b).
- Step 5 broken: the editor doesn't map `/cover.jpg` back to the post's folder for files that are already built. Existing covers would break on load.
- Step 6 picker opens in an empty folder, or a stored value other than `/other.jpg`: the `static`/`uploads` doubling from V4 happens with relative paths off too.

## V3 rerun against V5 (placeholder in `static`)

V3 is V5 with `static: content/blog/[full_slug]/` instead of `content/blog/v5/`.

| Step | Shows? | `src` | Picker folder | Stored value |
| --- | --- | --- | --- | --- |
| 9. V3 choose `other.jpg` | Can't choose | `/cover.jpg` (unchanged) | Unlabelled; one level up is the literal `content/blog/[full_slug]`. After clearing the image it opens at `content/blog/v3` (empty), whose parents are `content/blog/[full_slug]/content/blog` | — |
| 10. V3 upload | Yes, straight away | `https://app.cloudcannon.com/api/v0/editing_session_files/<id>/raw?…` | — | `/content/blog/v3/pexels-polina-tankilevitch.jpg` (save modal path the same) |

Confirms `static` doesn't fill in `[full_slug]`: the picker shows it literally, and the upload stores the full repo path where V5 stored `/pexels-….jpg`. Hugo's `.Resources.Get` can't find that value.

The upload still previewed, though `static` didn't match. Uploads previewed in every relative-off variant (Control, V3, V5) and in no relative-on variant (V1, V4). Upload preview may depend on the value being root-style rather than on `static`.

## V5 on the blog list

| Step | Shows? | Stored value |
| --- | --- | --- |
| 8. Blog list, V5 card, choose `other.jpg` | Yes | `/other.jpg` |

With no placeholder and relative paths off, an image region bound with `@file` on another page edits a bundle image correctly. Only the image on load breaks, as on V5's own page.

## Empty uploads variant (V6)

Question: with `uploads: ""` and relative paths on, does CloudCannon upload into the folder of the file being edited?

V6 is V1 with `uploads: ""` instead of `content/blog/[full_slug]/`. The docs say `uploads_use_relative_path` makes the stored value relative to the file being edited, and that `uploads` defaults to `uploads`; they don't say what an empty `uploads` means.

Steps. Copy each value straight away, then discard:

11. **Upload on V6's own page (`/blog/v6/`).** Click the cover and upload a new image. Where does the picker open? What's the stored value? Which folder does the save modal show the file going to?
12. **Upload from the blog list (`/blog/`).** Click V6's card cover (it's broken on load, as all bundle covers are there) and upload a new image. Same three questions.

| Step | Picker folder | Stored value | File saved to |
| --- | --- | --- | --- |
| 11. V6 own page | | | |
| 12. V6 from blog list | | | |

What the results mean:

- Step 11 saves to `content/blog/v6/` with a bare filename: an empty `uploads` means the file's own folder. Same result as V1 with no placeholder.
- Step 11 saves to `uploads/` or the repo root, with a `../`-style value: an empty `uploads` falls back to a default. Hugo can't find the value.
- Step 12 saves to `content/blog/v6/` with a bare filename: "the file being edited" is the post, so V6 also fixes uploads from other pages (#20a).
- Step 12 saves to `content/blog/` or elsewhere: it uses the page being edited, as #20a describes.
