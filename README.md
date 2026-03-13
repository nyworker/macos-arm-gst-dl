# GStreamer 1.28.0 from Source (macOS Apple Silicon)
[ should check the swift build new version: https://github.com/nyworker/macos-arm-gst-decklink-ndi ] 

Build of GStreamer with Rust plugins (gst-plugins-rs) and Decklink support.

Installed to `/usr/local/gstreamer`. 276 plugins, 1523 features.

## Prerequisites

- macOS on Apple Silicon with Homebrew
- Xcode Command Line Tools (`xcode-select --install`)

## 1. Install Build Tools

```bash
brew install meson ninja nasm bison
```

## 2. Install Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
cargo install cargo-c
```

Do NOT use `brew install rust` — use rustup. `cargo-c` is required for building gst-plugins-rs.

## 3. Install Library Dependencies

```bash
brew install glib gobject-introspection cairo pango graphene orc \
  json-glib libnice libsoup libsodium openssl@3 \
  x264 x265 aom dav1d svt-av1 libvpx opus faac faad2 fdk-aac \
  lame mpg123 flac libvorbis libogg theora speex \
  libpng jpeg-turbo librsvg libass webp openjpeg openexr imath little-cms2 \
  srt srtp rtmpdump libshout libsndfile \
  ffmpeg gtk+3 gtk4 gdk-pixbuf taglib harfbuzz gettext
```

## 4. Unlink Homebrew GStreamer

```bash
brew unlink gstreamer
```

This removes symlinks but keeps the Cellar intact for easy re-linking later.

## 5. Clone Repositories

```bash
cd /Users/txadmin/gst-dl

git clone --branch 1.28.0 --depth 1 \
  https://gitlab.freedesktop.org/gstreamer/gstreamer.git

git clone --branch gstreamer-1.28.0 --depth 1 \
  https://gitlab.freedesktop.org/gstreamer/gst-plugins-rs.git \
  gstreamer/subprojects/gst-plugins-rs
```

The `gst-plugins-rs` directory name must be exact — the monorepo's meson.build expects it.

## 6. Configure

```bash
cd /Users/txadmin/gst-dl/gstreamer

export PATH="/opt/homebrew/opt/bison/bin:/opt/homebrew/bin:$HOME/.cargo/bin:$PATH"
export PKG_CONFIG_PATH="/opt/homebrew/lib/pkgconfig:/opt/homebrew/share/pkgconfig"
export OPENSSL_NO_VENDOR=1
export OPENSSL_DIR="$(brew --prefix openssl@3)"

meson setup build \
  --prefix=/usr/local/gstreamer \
  --buildtype=release \
  --strip \
  -Dbase=enabled \
  -Dgood=enabled \
  -Dugly=enabled \
  -Dbad=enabled \
  -Dlibav=enabled \
  -Drs=enabled \
  -Ddevtools=enabled \
  -Dges=enabled \
  -Drtsp_server=enabled \
  -Dgpl=enabled \
  -Dtls=enabled \
  -Dtools=enabled \
  -Dorc=enabled \
  -Dorc-source=system \
  -Dbuild-tools-source=system \
  -Dtests=disabled \
  -Dexamples=disabled \
  -Ddoc=disabled \
  -Dgtk_doc=disabled \
  -Dqt5=disabled \
  -Dintrospection=enabled \
  -Dpython=disabled \
  -Dgst-plugins-bad:decklink=enabled \
  -Dgst-plugins-bad:opencv=disabled \
  -Dgst-plugins-rs:closedcaption=enabled \
  -Dgst-plugins-rs:dav1d=enabled \
  -Dgst-plugins-rs:sodium=enabled \
  -Dgst-plugins-rs:csound=disabled \
  -Dgst-plugins-rs:gtk4=enabled \
  -Dgst-plugins-rs:sodium-source=system \
  -Dlibnice=disabled
```

## 7. Build and Install

```bash
ninja -C build -j8

sudo mkdir -p /usr/local/gstreamer
sudo chown -R $(whoami):staff /usr/local/gstreamer
ninja -C build install
```

First build takes ~30-45 minutes (Rust crate downloads + compilation).

## 8. Use

```bash
source /Users/txadmin/gst-dl/gst-env.sh
```

Add to `~/.zshrc` to make permanent.

### Verify

```bash
gst-launch-1.0 --version        # 1.28.0
gst-inspect-1.0 decklink         # Decklink plugin elements
gst-inspect-1.0 rsfile           # Rust plugin check
gst-inspect-1.0 | tail -1        # Total plugin count
```

## Revert to Homebrew GStreamer

```bash
brew link gstreamer
```

## Notes

- The Decklink SDK headers (v12.2.2) are bundled in `subprojects/gst-plugins-bad/sys/decklink/osx/`. No external SDK path is needed.
- GTK3 + GTK4 ObjC class conflict warnings at runtime are cosmetic and harmless.
- To reconfigure, run `meson setup build --reconfigure` with the desired flags.
- To rebuild after source changes: `ninja -C build && ninja -C build install`.

- TODO: fully testing on txmini-2 but ffmpeg-dl with latest versions might be better
- 
