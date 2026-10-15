# Image Metadata Converter

A single-file browser tool that converts ComfyUI and Automatic1111 images to WebP without losing their generation metadata: the ComfyUI workflow and prompt, or the A1111 `parameters` text.

Open `png-to-webp-metadata.html` in Chrome or Edge. Nothing to install, and images never leave your computer.

## Features

- **Convert to WebP**: PNG, WebP, JPG, AVIF and EXR in; WebP out.
  - Lossless (pixel-identical) or lossy with a quality slider, plus an effort setting.
  - Encoded with Google's libwebp compiled to WebAssembly.
  - Output name format with `{name}` and `{q}` tokens, a "Saves as" preview, and warnings for name clashes.
- **Add metadata to existing WebPs**: for WebPs made with another converter. Drop the originals and the WebPs together; they're paired by file name and the metadata is copied over without re-encoding the image.
- Drag and drop files or folders, thumbnails, per-file remove, save to a folder (keeps subfolders) or download as ZIP.

## How the metadata is stored

The WebP gets an EXIF chunk in the same layout each app writes itself, so both can read it back:

| App | EXIF field | Content |
|---|---|---|
| ComfyUI | IFD0 `Model` (0x0110) | `prompt:{…}` |
| ComfyUI | IFD0 `Make` (0x010F) | `workflow:{…}` |
| A1111 / Forge | Exif `UserComment` (0x9286) | `UNICODE\0` + UTF-16BE parameters text |

Metadata is read from PNG text chunks (tEXt/zTXt/iTXt), the EXIF in JPG, AVIF and WebP, and the string attributes of EXR headers.

## Notes

- **Internet:** the first conversion downloads the encoder and decoders from jsDelivr: `@jsquash/webp` for WebP, and `three` for its EXR loader.
- **EXR:** converted from linear light to 8-bit sRGB. Values above white are clipped.
- **Unsupported:** animated WebPs.
- **Browser:** "Save to folder" uses the File System Access API (Chrome/Edge). Other browsers can use the ZIP download.

## Samples

`Sample/` holds test images from ComfyUI and A1111 in each supported format.
