# StreamGaffer third-party source

This repository holds the README and the file list. The source files themselves are attached to the release (they are too large for the repository):

**Release page:** [SOURCE ADDRESS TO BE INSERTED WHEN THE RELEASE EXISTS]

It is the corresponding source for the FFmpeg `ffprobe` program and the FFmpeg shared libraries that are distributed with the StreamGaffer Windows installer. It contains no StreamGaffer program code.

## What was distributed

The installer carries `ffprobe.exe` and these libraries: `avcodec-63.dll`, `avdevice-63.dll`, `avfilter-12.dll`, `avformat-63.dll`, `avutil-61.dll`, `swresample-7.dll`, `swscale-10.dll`. They are built from FFmpeg's `n9.0.2` tag, unmodified, with the BtbN FFmpeg-Builds scripts at commit `3e6685eda92f9288c15ac320139622dcedca09a4`, using 74 one-line edits that switch off every library `ffprobe` does not need to read a file (see "Patches"). The archive is `ffmpeg-n9.0.2-win64-lgpl-shared-9.0-probe.zip`, and the program reports `ffprobe version n9.0.2-20261006`.

- FFmpeg's own code is under the GNU Lesser General Public License, version 3 or later. It was configured with `--enable-version3` and without `--enable-gpl` or `--enable-nonfree`.
- The third-party libraries linked in are dav1d (BSD 2-Clause), zlib, GNU libiconv (LGPL 2.1 or later), liblzma from xz, and libxml2 (MIT), plus Windows' own Schannel and Universal C Runtime, which Windows supplies.
- The compiler runtime (libgcc, under the GNU GPL version 3 with the GCC Runtime Library Exception 3.1), the mingw-w64 runtime and winpthreads are linked in statically.

## Files in the release

Release assets are a flat list; the names below are the file names.

| Files | What they are |
|---|---|
| `ffmpeg-n9.0.2-win64-lgpl-shared-9.0-probe.zip` | The archive that was bundled. SHA-256 `13d80de29c4d3993fbfeb2b608a90e1d85a150030627ae34b6cf472f7f0d4b87`. |
| `bundle.json`, `configure-line.txt`, `ffprobe-version-and-licence-output.txt`, `file-hashes-of-the-nine-files.txt` | The build record, the exact configure line, the output of `ffprobe -version`, `-L` and `-buildconf`, and the size and SHA-256 of each bundled file. `bundle.json` is a record of the build; it is not the source. |
| `ffmpeg-9.0.2.tar.xz`, `.asc`, `ffmpeg-devel.asc` | The FFmpeg 9.0.2 source, its signature, and FFmpeg's release signing key (fingerprint `FCF9 86EA 15E6 E293 A564 4F10 B432 2F04 D676 58D8`). SHA-256 `8c3850283eb25fa026482078a04051e0be17347b09ef81a0849bec15a96e002e`. |
| `btbn-ffmpeg-builds-3e6685eda92f.tar.gz` | The BtbN FFmpeg-Builds scripts, unmodified, at the commit above. |
| `ffprobe-edits.patch`, `ffprobe-edits-stat.txt` | Our 74 one-line edits to those scripts, as a `git diff`, and its summary. |
| `btbn-recipe-as-built.tar.gz` | The scripts, variant file and Dockerfiles exactly as they were when the program was built (the BtbN scripts with the edits applied). |
| `final-image-facts.txt`, `libraries-actually-linked.txt`, `toolchain-notes.txt`, `build.log.gz` | The build image digests, the libraries linked into the program with the commit each script pins, the compiler toolchain versions and where its tarballs came from, and the full build log. |
| `20-zlib-…`, `20-libiconv-…` (two archives, libiconv and its gnulib), `25-xz-…`, `25-libxml2-…`, `50-dav1d-…` | The source of each linked library at the exact commit the BtbN scripts pin (the number is the build-script order, then the library, then the first 12 characters of the commit). Submodules are included; `.git` folders are not. |
| `10-mingw-…`, `10-mingw-std-threads-…` | The mingw-w64 source at the commit the build script pins (it includes winpthreads), and mingw-std-threads. |
| `COPYING.RUNTIME`, `COPYING3` | The GCC Runtime Library Exception 3.1 and the GPL version 3, from the GCC 16.2.0 source. |

`MANIFEST-SHA256.txt` in this repository lists every release file with its SHA-256 and size.

## How to check it

1. Check the FFmpeg tarball against the hash above, then `gpg --import ffmpeg-devel.asc` and `gpg --verify ffmpeg-9.0.2.tar.xz.asc ffmpeg-9.0.2.tar.xz`.
2. FFmpeg tag `n9.0.2` points at commit `946fcce07b6dcd0331c8cc609192aeff5e1924f8`.
3. Check every other file against `MANIFEST-SHA256.txt`.
4. To rebuild: take the BtbN scripts at the commit above, apply `ffprobe-edits.patch`, and run BtbN's `makeimage.sh win64 lgpl-shared 9.0` and then `build.sh win64 lgpl-shared 9.0` with `GIT_BRANCH_OVERRIDE=n9.0.2`, as described in BtbN's own documentation. The compiler toolchain is built inside BtbN's Docker image from the tarballs listed in `toolchain-notes.txt`; its Dockerfiles do not pin every version, so a later rebuild may report a different compiler build date.

## Patches

FFmpeg itself is built unmodified from the `n9.0.2` tag. The only change to the BtbN scripts is that in 74 of their stage scripts the line `return 0` inside `ffbuild_enabled()` is changed to `return -1`, so that the library is not built. The patch file lists every one. None of the libraries that remain in the build has a patch file. One of the remaining stage scripts, the mingw-w64 one, edits one of its own source files while it builds: a `sed` command in `scripts.d/10-mingw.sh` gives the constructor in `mingw-w64-crt/ssp/stack_chk_guard.c` the highest priority. It is in the script, which is in `btbn-ffmpeg-builds-3e6685eda92f.tar.gz`.

## What is not included

- The compiler toolchain itself (gcc 16.2.0, binutils, crosstool-NG 1.29.0.7_b1a94f6), which comes from BtbN's build image. Its source tarballs are public at the versions and addresses in `toolchain-notes.txt`. The mingw-w64 source, which includes winpthreads, is included.
- Libraries that the build configuration disables. They are not in the program.

Question or missing file: open an issue in this repository.
