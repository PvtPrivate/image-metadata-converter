# Image Metadata Converter

A single-file browser tool that converts ComfyUI and Automatic1111 images between WebP, AVIF, JPEG XL, JPEG and PNG without losing their generation metadata: the ComfyUI workflow and prompt, or the A1111 `parameters` text.

Open `index.html` in Chrome or Edge. Nothing to install, and images never leave your computer.

## Features

- **Convert**: PNG, WebP, JPG, AVIF, JPEG XL and EXR in; WebP, AVIF, JPEG XL, JPEG or PNG out.

  | Output | Modes | Encoder | Options |
  |---|---|---|---|
  | WebP | lossless or lossy | libwebp | quality, effort, sharp YUV |
  | AVIF | lossless or lossy | libavif (aom) | quality, effort (encoder speed), sharp YUV |
  | JPEG XL | lossless or lossy | libjxl | quality, effort |
  | JPEG | lossy | MozJPEG | quality, background colour for transparent areas |
  | PNG | lossless | oxipng | effort (optimisation level) |

  - Output name format with `{name}` and `{q}` tokens, a "Saves as" preview, and warnings when names clash or would overwrite the source.
- **Restore metadata**: for images converted or edited with another tool that dropped the metadata.
  - Two lists: **Originals** (any input format) and **Files to fix** (WebP, PNG, JPEG, AVIF or JPEG XL).
  - Files to fix are matched to originals by name. A dropdown lets you pick a different original, or one for a file that has no match.
  - Only the metadata is written. The image isn't re-encoded, and existing generation metadata is replaced, not duplicated.
- Drag and drop files or folders, thumbnails, per-file remove, save to a folder (keeps subfolders) or download as ZIP.

## How the metadata is stored

Each output format gets the metadata in the layout the apps themselves use, so ComfyUI and A1111 can read it back.

**WebP, AVIF, JPEG XL and JPEG** use EXIF:

| App | EXIF field | Content |
|---|---|---|
| ComfyUI | IFD0 `Model` (0x0110) | `prompt:{…}` |
| ComfyUI | IFD0 `Make` (0x010F) | `workflow:{…}` |
| A1111 / Forge | Exif `UserComment` (0x9286) | `UNICODE\0` + UTF-16BE parameters text |

- **WebP:** an `EXIF` chunk.
- **AVIF:** an `Exif` item linked to the primary image with a `cdsc` reference.
- **JPEG XL:** an `Exif` box in the JXL container, right after `ftyp`. The encoder writes a bare codestream, so it is wrapped in a container (`JXL `, `ftyp`, `jxlc`) first. Brotli-compressed (`brob`) Exif boxes are read where the browser supports Brotli decompression.
- **JPEG:** an APP1 segment. JPEG caps it at 64 KB, so a bigger workflow fails with an error instead of being dropped.

**PNG** uses one text chunk per key (`prompt`, `workflow`, `parameters`): `tEXt` when the text fits Latin-1, otherwise UTF-8 `iTXt`. This is what Pillow writes.

Metadata is read from PNG text chunks, the EXIF in JPG, AVIF, JPEG XL and WebP, and the string attributes of EXR headers.

## Notes

- **Internet:** each encoder and decoder downloads from jsDelivr the first time it's used:
  - `@jsquash/webp`, `@jsquash/avif`, `@jsquash/jxl`, `@jsquash/jpeg` and `@jsquash/oxipng` for the formats (JPEG XL is also decoded with `@jsquash/jxl`, since most browsers can't).
  - `three` for its EXR loader.

  AVIF and JPEG XL use the single-threaded encoders, because the multi-threaded ones need cross-origin isolation, which a page opened from disk can't have.
- **EXR:** converted from linear light to 8-bit sRGB. Values above white are clipped.
- **Unsupported:** animated WebPs.
- **Browser:** "Save to folder" uses the File System Access API (Chrome/Edge). Other browsers can use the ZIP download.

## Samples

`Sample/` holds test images from ComfyUI and A1111 in each supported format.
