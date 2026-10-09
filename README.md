# Hugo bundle images in CloudCannon

How CloudCannon image inputs and image regions handle Hugo page-bundle images: `content/blog/<post>/cover.jpg`, read with `.Resources.Get`. Each post `/blog/vN/` is its own collection, so each has its own `cover` input `paths`. Tested on a hosted site, 2026-10-08/09.

- [TESTING.md](TESTING.md): how to run each check, and the raw values.

## The `static` problem

A bundle's folder differs per post, so `paths.static` would need a placeholder: `static: content/blog/[full_slug]/`. `uploads` fills in placeholders, but `static` doesn't; `[full_slug]` stays literal.

V3 and V5 differ only in that placeholder:

| | `static` | Choose existing | Upload stores | Upload previews |
| --- | --- | --- | --- | --- |
| V3 | `content/blog/[full_slug]/` | ❌ browser opens under a literal `[full_slug]` folder | `/content/blog/v3/x.jpg` (Hugo can't find it) | ✅ |
| V5 | `content/blog/v5/` | ✅ stores `/other.jpg` | `/x.jpg` (`.Resources.Get "/x.jpg"` finds it) | ✅ |

Filling in placeholders would make V3 behave like V5. Two more things stand between that and fully working bundle images:

1. **First load.** An image region sets `src` to the raw stored value on load (`editable-regions` `nodes/editable-image.ts:136`), so V5's `/cover.jpg` breaks until it's edited. After an edit, the same `getPreviewUrl` call resolves through `static` to a CloudCannon file URL.
2. **Other pages.** Placeholders are filled from the page being edited, not the `@file` target. Uploading V10's cover from the blog list saves to `content/blog/blog/`.

## Results

`uploads` is `content/blog/[full_slug]/` unless shown otherwise. "Shows later" means the upload saved to the right folder with a value Hugo can read, but only appears on the page after the next build (confirmed on V1).

| Post | `static` | Relative | Load | Choose | Upload |
| --- | --- | --- | --- | --- | --- |
| V1 | `""` | on | ✅ own page | ✅ | ✅ shows later |
| V2 | `content/blog/[full_slug]/` | on | ✅ | ❌ empty folder | ✅ shows later |
| V3 | `content/blog/[full_slug]/` | off | ❌ | ❌ | ❌ full repo path |
| V4 | `content/blog/v4/` | on | ✅ | ⚠️ starts in empty folder | ❌ `../../../x.jpg` |
| V5 | `content/blog/v5/` | off | ❌ | ✅ | ✅ previews |
| V6 | `""`, `uploads: ""` | on | — | ✅ | ❌ repo root |
| V7 | left out, `uploads` left out | on | ✅ | ✅ | ❌ `/uploads/` |
| V8 | left out | on | ✅ own page | ✅ | ✅ shows later |
| V9 | left out | off | ✅ | ❌ full repo path | ❌ full repo path |
| V10 | `content` | off | ✅ any page | ✅ any page | ✅ previews (own page only) |
| V11 | `content` | on | ✅ | ⚠️ starts in empty folder | ❌ `../../../blog/v11/x.jpg` |

## Findings

- **On load, an image region shows the raw stored value.** It only shows if that value already works as a URL from the current page. `cover.jpg` works only on the post's own page.
- **`uploads_use_relative_path` decides how the value is written, not where the file goes.**
  - On: relative to the edited file (`cover.jpg`).
  - Off: relative to `static` (`/cover.jpg`).
- **`uploads` decides where the file goes.** `""` is the repo root; left out, it's `uploads/`.
- **Relative paths on need `static` to be `""` or left out.** Any other value opens the image browser at `static` + `uploads`, a folder that doesn't exist (V2, V4, V11). V4 and V11 also store broken `../` values.
- **Only root-style values (`/…`) preview after a change,** and they do even when `static` is wrong (V3, V9). Relative values show only after the next build.
- **V10 is a workaround that works today.** It stores the image's site URL. It needs a template that removes the post's URL before `.Resources.Get`, and a URL that matches the folder.
- **The editor's Hugo can't see bundle files.** A component region can still show a bundle image on another page by building the URL from the post's `.RelPermalink` (home page).

## Reading the site

- **Post page:** the cover input's `paths` (read from `cloudcannon.config.yml`), what it tests, and the result. Below that:
  - **Control:** a `static/` image. If it's broken, the page is broken; ignore its cover.
  - **Cover:** the bundle image, in an image region.
- **Blog list:** every cover again, in image regions bound with `@file`.
- **Home page:** V1's cover in a component region (`layouts/partials/bundle-cover.html`). The text under it shows whether the `src` came from `.Resources.Get` or was built from the permalink.

All files open in the Visual Editor by default.
