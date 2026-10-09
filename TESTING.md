# Testing

## Setup

Push to a repo and connect it to a new CloudCannon site. `.cloudcannon/initial-site-settings.json` sets the build (`hugo mod vendor && hugo`, Hugo 0.166.0).

Each post starts with `cover.jpg` and `other.jpg` (the same image, flipped) in its folder. If a post's Control box is broken, the page is broken; fix it before reading the cover.

## Steps

On each post's own page, in the Visual Editor. Copy each value straight away, then discard. An unsaved upload stops showing after a page refresh.

1. **Load.** Don't touch anything. Does the cover show? Inspect the `<img>` and copy its `src`.
2. **Choose.** Click the cover. Note which folder CloudCannon's image browser opens in, and whether `cover.jpg` and `other.jpg` are there. Choose `other.jpg`. Does it show? Copy its `src` and the stored value.
3. **Upload.** Click the cover and upload a new image. Does it show straight away? Copy the stored value and the folder in the save modal.

For a variant whose own-page results are good, repeat the steps from the blog list.

## Results: own page

| Post | Load (`src`) | Choose: browser opens in | Choose: value (`src`) | Upload: shows? | Upload: value | Upload: saved to |
| --- | --- | --- | --- | --- | --- | --- |
| V1 | ✅ `cover.jpg` | The bundle | ✅ `other.jpg` | No (shows after save and rebuild) | `send-it-ai.png` | `content/blog/v1/` |
| V2 | ✅ `cover.jpg` | `content/blog/[full_slug]/content/blog/v2` (breadcrumb), empty | Can't choose | No | `screenshot-….png` | `content/blog/v2/` |
| V3 | ❌ `/cover.jpg` | Under a literal `content/blog/[full_slug]`, empty | Can't choose | Yes, `editing_session_files` URL | `/content/blog/v3/pexels-….jpg` | `content/blog/v3/` |
| V4 | ✅ `cover.jpg` | Breadcrumb `content/blog/v4`, empty | ✅ `other.jpg`, after going back to `blog > v4` | No | `../../../pexels-….jpg` | `content/blog/v4/` |
| V5 | ❌ `/cover.jpg` | The bundle | ✅ `/other.jpg` (`…/files/content%2Fblog%2Fv5%2Fother.jpg`) | Yes, `editing_session_files` URL | `/pexels-….jpg` | `content/blog/v5/` |
| V6 | not recorded | The bundle | ✅ `other.jpg` | No | `../../../pexels-….jpg` | Repo root |
| V7 | ✅ `cover.jpg` | The bundle | ✅ `other.jpg` | No | `../../../uploads/screenshot-….png` | `/uploads/` |
| V8 | ✅ `cover.jpg` | The bundle | ✅ `other.jpg` | No | `screenshot-….png` | `content/blog/v8/` |
| V9 | ✅ `cover.jpg` | not recorded | ✅ `/content/blog/v9/other.jpg` (`…/files/…` URL) | Yes | `/content/blog/v9/screenshot-….png` | `content/blog/v9/` |
| V10 | ✅ `/blog/v10/cover.jpg` | not recorded | ✅ `/blog/v10/other.jpg` | Yes, `editing_session_files` URL | `/blog/v10/pexels-….jpg` | `content/blog/v10/` |
| V11 | ✅ `cover.jpg` | Breadcrumb `content/blog/v11`, empty | ✅ `other.jpg`, after going back to `blog > v11` | No | `../../../blog/v11/screenshot-….png` | `content/blog/v11/` |

**Control box (`static: static`, relative off):** an upload previews straight away with a plain `/images/<file>` `src`, and stops showing after a refresh without saving.

## Results: other pages

| Where | Post | Load | Choose | Upload |
| --- | --- | --- | --- | --- |
| Blog list, image region | V1 (2026-10-08) | ❌ | ❌ stores `<post>/<file>`, relative to the list page | not tested |
| Blog list, image region | V5 | ❌ | ✅ `/other.jpg` | not tested |
| Blog list, image region | V10 | ✅ `/blog/v10/cover.jpg` | ✅ `/blog/v10/other.jpg`, files API `src` | Shows, but stores `/blog/blog/<file>` and saves to `content/blog/blog/` |
| Home, component region | V1 | ✅ | ✅ `other.jpg`; `src` `/blog/v1/other.jpg`, `via computed`, `renderer editor` | Value and folder right (`send-it-ai.png`, `content/blog/v1/`); shows after save and rebuild |

## Notes

These are readings of the results above, not separately tested.

- **Relative on with a non-empty `static` (V2, V4, V11):** the image browser opens at `static` + `uploads` (`content/content/blog/v11/`), even when the breadcrumb shows the right path. V4 and V11 work out the relative value from that doubled folder: three levels up from `content/content/blog/v11/` is `content/`, then `blog/v11/<file>`.
- **V2's upload value is right, unlike V4 and V11.** Possibly the value only goes wrong when `static` is a prefix of `uploads`, and V2's literal `[full_slug]` never matches.
- **V2's upload showed in the input's thumbnail but not on the page.** Other variants' thumbnails weren't checked.
- **V10 from the blog list:** `[full_slug]` was filled from `content/blog/_index.md` (`blog`), not from V10.
- **Home component:** `via computed` in the editor means `.Resources.Get` found nothing there, so the editor's Hugo doesn't have bundle files.
