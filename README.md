# Lagrange

Lagrange is a desktop GUI client for browsing [Geminispace](https://gemini.circumlunar.space/). It offers modern conveniences familiar from web browsers, such as smooth scrolling, inline image viewing, multiple tabs, visual themes, Unicode fonts, bookmarks, history, and page outlines.

Like Gemini, Lagrange has been designed with minimalism in mind. It depends on a small number of essential libraries. It is written in C and uses [SDL](https://libsdl.org/) for hardware-accelerated graphics. [OpenSSL](https://openssl.org/) is used for secure communications.

![Lagrange window open on URL "about:lagrange"](lagrange_about.png)

## Features

* Beautiful typography using Unicode fonts
* Autogenerated page style and Unicode icon for each Gemini domain
* Smart suggestions when typing the URL — search bookmarks, history, identities
* Sidebar for page outline, managing bookmarks and identities, and viewing history
* Multiple tabs
* Identity management — create and use TLS client certificates
* Audio playback: MP3, Ogg Vorbis, WAV
* And more! Open `about:help` in the app, or see [help.gmi](https://git.skyjake.fi/gemini/lagrange/raw/branch/release/res/about/help.gmi)

## Downloads

Prebuilt binaries for Windows, macOS and Linux can be found in [Releases][rel]. You can also find [Lagrange on Flathub for Linux](https://flathub.org/apps/details/fi.skyjake.Lagrange).

On macOS you can install and upgrade via Homebrew:

```
brew install --cask lagrange
```

On openSUSE Tumbleweed:

```
sudo zypper install lagrange
```

## How to compile

This is how to build Lagrange in a POSIX-compatible environment. The required tools are a C11 compiler (e.g., Clang or GCC), CMake and `pkg-config`.

1. Download and extract a source tarball from [Releases][rel]. Please note that the GitHub/Gitea-generated tarballs do not contain the ["the_Foundation" submodule](https://git.skyjake.fi/skyjake/the_Foundation); check which tarball you are getting. Alternatively, you may also clone the repository and its submodules: `git clone --recursive --branch release https://git.skyjake.fi/gemini/lagrange`
2. Check that you have the required dependencies installed: CMake, SDL 2, OpenSSL 1.1.1, libpcre, zlib, libunistring. For example, on macOS this would do the trick (using Homebrew): ```brew install cmake sdl2 openssl@1.1 pcre libunistring``` Or on Ubuntu: ```sudo apt install cmake libsdl2-dev libssl-dev libpcre3-dev zlib1g-dev libunistring-dev```
3. Optionally, install the mpg123 decoder library for MPEG audio support. For example, the macOS Homebrew package is `mpg123` and on Ubuntu it is `libmpg123-dev`.
4. Create a build directory.
5. In your empty build directory, run CMake: ```cmake {path_of_lagrange_sources} -DCMAKE_BUILD_TYPE=Release```
6. Build it: ```cmake --build .```
7. Now you can run `lagrange`, `lagrange.exe`, or `Lagrange.app`.

### Unicode text rendering

(details about HarfBuzz and FriBidi)

### Installing to a directory

Set `CMAKE_INSTALL_PREFIX` to install to a directory of your choosing.

1. `cmake {path_of_lagrange_sources} -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/dest/path`
2. `cmake --build . --target install`

This will also install an XDG .desktop file for launching the app.

### Build options

| CMake Option | Description |
| ------------ | ----------- |
| `ENABLE_BINCAT_SH` | Merge resource files (fonts, etc.) together using a Bash shell script. By default this is **OFF**, so _res/bincat.c_ is compiled as a native executable for this purpose. However, when cross-compiling, native binaries built during the CMake run may be targeted for the wrong architecture. Set this to **ON** if you are having problems with bincat while running CMake. |
| `ENABLE_IDLE_SLEEP` | Sleep in the main thread instead of waiting for events. On some platforms, `SDL_WaitEvent()` may have a relatively high CPU usage. Setting this to **ON** polls for events periodically but otherwise keeps the main thread sleeping, reducing CPU usage. The drawback is that there is a slightly increased latency reacting to new events after idle mode ends. |
| `ENABLE_HARFBUZZ` | Use the HarfBuzz library for shaping Unicode text. This is required for correctly rendering complex scripts and composite glyphs. If disabled, a simplified text shaping algorithm is used that only works for left-to-right languages like English. |
| `ENABLE_KERNING` | Use kerning information in the fonts to adjust glyph placement. Setting this **ON** improves text appearance in subtle ways but slows down text rendering. It may be a good idea to set this to **OFF** when running on a slow CPU. |
| `ENABLE_MPG123` | Use the mpg123 library for decoding MPEG audio files. |
| `ENABLE_RESOURCE_EMBED` | Embed all resource files into the Lagrange executable instead of keeping them in a separate file that gets loaded at launch. Setting this **ON** makes it much slower to run CMake and to compile Lagrange. |
| `ENABLE_WINDOWPOS_FIX` | Set correct window position after the window has already been shown. This may be necessary on some platforms to prevent the window from being restored to the wrong position. |
| `ENABLE_X11_SWRENDER` | Default to software rendering when running under X11. By default Lagrange attempts to use the GPU for rendering the user interface. You can also use the `--sw` option at launch to force software rendering. |

### Compiling on macOS

When using OpenSSL 1.1.1 from Homebrew, you must add its pkgconfig path to your `PKG_CONFIG_PATH` environment variable, for example:

    export PKG_CONFIG_PATH=/opt/homebrew/Cellar/openssl@1.1/1.1.1i/lib/pkgconfig

Also, SDL's trackpad scrolling behavior on macOS is not optimal for regular GUI apps because it emulates a physical mouse wheel. This may change in a future release of SDL, but at least in 2.0.14 (and earlier) a [small patch](https://git.skyjake.fi/gemini/lagrange/raw/branch/dev/sdl2-macos-ios.diff) is required to allow momentum scrolling to come through as single-pixel mouse wheel events. Note that SDL comes with an Xcode project; use the "Shared Library" target and check that you are doing a Release build.

### Compiling on Windows

Windows builds require [MSYS2](https://www.msys2.org). In theory, [Clang](https://clang.llvm.org/docs/MSVCCompatibility.html) or GCC (on [MinGW](http://mingw.org)) could be set up natively on Windows for compiling everything, but the_Foundation still lacks Win32 implementations for the Socket and Process classes and these are required by Lagrange. [Cygwin](http://cygwin.org) is a possible alternative to MSYS2, although Cygwin builds have not been tested.

You should use a version of the SDL 2 library that is compiled for native Windows (i.e., the MSVC variant) instead of the version from MSYS2 or MinGW. You can download a copy of the SDL binaries from [libsdl.org](https://libsdl.org/). To make configuration easier in your MSYS2 environment, consider writing a custom sdl2.pc file so `pkg-config` can automatically find the correct version of SDL. Below is an example of what your sdl2.pc might look like:

```
prefix=/c/SDK/SDL2-2.0.12/
arch=x64
libdir=${prefix}/lib/${arch}/
incdir=${prefix}/include/

Name: sdl2
Description: Simple DirectMedia Layer
Version: 2.0.12-msvc
Libs: ${libdir}/SDL2.dll -mwindows
Cflags: -I${incdir}
```

The *-mwindows* option is particularly important as that specifies the target is a GUI application. Also note that you are linking directly against the Windows DLL — do not use any prebuilt .lib files if available, as those as specific to MSVC.

`pkg-config` will find your .pc file if it is on `PKG_CONFIG_PATH` or you place it in a system-wide pkgconfig directory.

Once you have compiled a working binary under MSYS2, there is still an additional step required to allow running it directly from the Windows shell: the shared libraries from MSYS2 must be found either via `PATH` or by copying them to the same directory where `lagrange.exe` is located.

### Compiling on Raspberry Pi

On Raspberry Pi 4/400, you can compile and run Lagrange just like on a regular desktop PC. Accelerated OpenGL graphics should work fine under X11.

On Raspberry Pi 3 or earlier, you should use a version of SDL that is compiled to take advantage of the Broadcom VideoCore OpenGL ES hardware. This provides the best performance when running Lagrange in a console. OpenGL under X11 on Raspberry Pi 2/3 is quite 
slow/experimental. When running under X11, software rendering is the best choice and the SDL from Raspbian etc. is sufficient.

The following build options are recommended on Raspberry Pi 2/3:

* `ENABLE_KERNING=NO`: faster text rendering without noticeable loss of quality
* `ENABLE_WINDOWPOS_FIX=YES`: workaround for window position restore issues (SDL bug)
* `ENABLE_X11_SWRENDER=YES`: use software rendering under X11

[rel]: https://git.skyjake.fi/gemini/lagrange/releases
[tf]:  https://git.skyjake.fi/skyjake/the_Foundation

## User files

On Windows, user files are stored in `%HOMEPATH%/AppData/Roaming/fi.skyjake.Lagrange/`, unless one is using the portable distribution and there is a `userdata` subdirectory present in the executable directory.

On macOS, user files are stored in `~/Library/Application Support/fi.skyjake.Lagrange/`.

On Linux/*BSD/other operating systems, user files stored in `~/.config/lagrange` unless you have customized the XDG directories, in which case the `XDG_CONFIG_HOME` environment variable is used to determine where user files saved.

The usage and contents of the user files are described in the Help document. You can delete one or more of the files while Lagrange is not running to reset the corresponding data to the default/empty state.

One instance of Lagrange can be running at a time per user file directory.
