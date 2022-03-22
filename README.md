# WebP file format plug-in for Photoshop

Current plug-in version: WebPShop 0.4.2

WebPShop is a Photoshop module for opening and saving WebP images, including
animations.

**Important note: [Photoshop 23.2 and newer natively supports WebP](https://helpx.adobe.com/photoshop/kb/support-webp-image-format.html).**
However some features such as preview at encoding and animations are missing.
WebPShop can still be installed and used for these use cases and/or for
Photoshop 23.1 and below.

Please look at the files LICENSE and CONTRIBUTING in the "docs" folder before
using the contents of this repository or contributing.

## Installation

Download the binary at https://github.com/webmproject/WebPShop/releases. \
Direct link for Windows x64:
https://github.com/webmproject/WebPShop/releases/download/v0.4.0/WebPShop_0_4_0_Win_x64.8bi \
Direct link for MacOS (extract the ZIP archive afterwise):
https://github.com/webmproject/WebPShop/releases/download/v0.4.0/WebPShop_0_4_0_Mac_Universal.zip \
Move the plug-in (the .8bi binary for Windows or the .plugin folder for MacOS)
to the Photoshop plug-in directory
(`C:\Program Files\Common Files\Adobe\Plug-Ins\CC` for Windows,
`/Library/Application Support/Adobe/Plug-Ins/CC` for Mac). Run Photoshop.

On macOS 10.15+, the prompt "WebPShop.plugin cannot be opened because
the developer cannot be verified" can be bypassed by running the following
in Terminal (Finder > Applications > Utilities):

```
sudo xattr -r -d com.apple.quarantine /Library/Application Support/Adobe/Plug-Ins/CC/WebPShop.plugin
```

## Features

*   `Open`, `Open As` menu commands can be used to read .webp files.
*   `Save a Copy...` menu command can be used to write .webp files. Encoding
    parameters can be tuned through the UI.

![WebPShop encoding settings - Windows](docs/webpshop_enc_ui_windows.webp)

Photoshop 23.2 and above has partial native WebP support. It will appear as
`WebP (*.WEBP)` in the "Save as type:" drop-down list at encoding. WebPShop will
still appear as `WebPShop (*.WEBP, *.WEBP)`. Use the latter. \
WebPShop is used at decoding.

## Encoding settings

For information, the quality slider maps the following ranges to their internal
WebP counterparts (see SetWebPConfig() in WebPShopEncodeUtils.cpp):

    | Quality slider value   -> | 0    ...    97 | 98         99 |    100   |
    |---------------------------|----------------|---------------|----------|
    | WebP encoding settings -> | Lossy, quality | Near-lossless | Lossless |
    |                           | 0    ...   100 | 60         80 |          |

The radio buttons offer several levels of compression effort:

    | Label   | WebP speed setting  | Sharp YUV    | WebP "quality" setting |
    |         |                     | (lossy only) | (except for lossy)     |
    |---------|---------------------|--------------|------------------------|
    | Fastest |          1          |      No      |            0           |
    | Default |          4          |      No      |           75           |
    | Slowest |          6          |      Yes     |          100           |

## Limitations

*   Only English is currently supported.
*   Only "RGB Color" image mode is currently supported.
*   16 and 32 bits/channels are downscaled to 8 bits/channels before the WebP
    encoding because WebP only supports 8-bit internally.
    Exports from 32-bit documents should include the color profile in the WebP
    encoding settings, otherwise they might appear darker than expected.
*   WebP images cannot exceed 16383 x 16383 pixels.
*   The Timeline data is not used; thus animations rely on layers for defining
    frames (set duration as "(123 ms)" in each layer's name), and they need to
    be rasterized before saving.
*   On some images, lossless compression might produce smaller file sizes than
    lossy. That's why the quality slider is not linear. The same problem exists
    with the radio buttons controlling the compression effort.
*   The color profile is always applied to the Preview image on Windows and
    never applied on macOS, regardless of the related checkbox state.
*   This plug-in does not extend `Export As` neither `Save for Web`.
*   Encoding and decoding are done in a single pass. It is not currently
    possible to cancel such actions, and it might take some time on big images.
*   Only the latest Photoshop release is supported.

## Troubleshooting

If the plug-in is not detected or does not behave as expected, the steps below
might help:

*   Update Photoshop to the latest version.
*   Double-check that the plug-in binaries match the Operating System and the
    architecture.
*   The plug-in should be listed in the "Help > About Plugins" submenu if it is
    found by Photoshop.
*   If it is undetected, disable any antivirus program or allow the plug-in
    execution (including in MacOS and Windows built-in protections).
*   If it is still undetected, try each of these folders. Windows paths:

        C:\Program Files\Common Files\Adobe\Plug-Ins\CC
        C:\Program Files\Common Files\Adobe\Plug-Ins\CC\File Formats
        C:\Program Files\Adobe\Adobe Photoshop 2022\Plug-ins

    MacOS paths:

        /Library/Application Support/Adobe/Plug-Ins/CC
		Applications/Adobe Photoshop/Plug-ins/

*   If it is still undetected, remove all plug-ins from all folders and copy
    WebPShop in only one of these folders, in case there is a plug-in conflict.
    Restart the computer and/or Photoshop.

If the issue still occurs, check https://github.com/webmproject/WebPShop/issues
to see if it is already mentioned or open a new bug report otherwise.

## Software architecture

The `common` folder contains the following:

*   `WebPShop.h` is the main header, containing most functions.
*   `WebPShop.cpp` contains the plug-in entry point (called by host).
*   `WebPShop.r` and `WebPShopTerminology.h` represent the plug-in properties.
*   Functions in `WebPShopSelector*` are called in `WebPShop.cpp`.
*   `WebPShop*Utils.cpp` are helper functions.
*   `WebPShopScripting.cpp` is mostly used for automation.
*   `WebPShopUI*` display the encoding parameters window and the About box.

The `win` folder contains a Visual Studio solution and project, alongside with
`WebPShop.rc` which is the encoding parameters window layout and About box.

The `mac` folder contains an XCode project. `WebPShopUIDialog_mac.h` and `.mm`
describe the UI layout, while `WebPShopUI_mac.mm` handles the window events.

## Build

Current libwebp version: WebP 1.2.2

Use Microsoft Visual Studio (2019 and above) for Windows and XCode for Mac.

*   Download the latest Adobe Photoshop Plug-In and Connection SDK at
    https://console.adobe.io/downloads/ps,
*   Put the contents of this repository in a "webpshop" folder located at
    `adobe_photoshop_sdk_[version]/pluginsdk/samplecode/format`,
*   Download the latest WebP binaries at
    https://developers.google.com/speed/webp/docs/precompiled or build them,
*   Add `path/to/webp/includes` and `path/to/webp/includes/src` as Additional
    Include Directories to the WebPShop project [1],
*   Add webp, webpdemux, webpmux libraries as Additional Dependencies to the
    WebPShop project [1],
*   Build with the same architecture and configuration as your Photoshop
    installation and the WebP binaries (x64 or arm64, Debug/Release),
*   By example for Windows, it should output the plug-in file `WebPShop.8bi` in
    `adobe_photoshop_sdk_[version]/pluginsdk/samplecode/Output/Win/x64/Release`.

[1] By default the XCode project includes and links to the
`libwebp-[version]-mac-[version]` folder in the `webpshop` directory. The VS
project expects `libwebp-[version]-windows-x64` (or `-arm64`).
