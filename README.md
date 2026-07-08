<p align="center">
  <img src="res/logo-header.svg" alt="RustDesk - Your remote desktop"><br>
  <a href="#raw-steps-to-build">Build</a> •
  <a href="#how-to-build-with-docker">Docker</a> •
  <a href="#file-structure">Structure</a> •
  <a href="#snapshot">Snapshot</a><br>
  [<a href="docs/README-UA.md">Українська</a>] | [<a href="docs/README-CS.md">česky</a>] | [<a href="docs/README-ZH.md">中文</a>] | [<a href="docs/README-HU.md">Magyar</a>] | [<a href="docs/README-ES.md">Español</a>] | [<a href="docs/README-FA.md">فارسی</a>] | [<a href="docs/README-FR.md">Français</a>] | [<a href="docs/README-DE.md">Deutsch</a>] | [<a href="docs/README-PL.md">Polski</a>] | [<a href="docs/README-ID.md">Indonesian</a>] | [<a href="docs/README-FI.md">Suomi</a>] | [<a href="docs/README-ML.md">മലയാളം</a>] | [<a href="docs/README-JP.md">日本語</a>] | [<a href="docs/README-NL.md">Nederlands</a>] | [<a href="docs/README-IT.md">Italiano</a>] | [<a href="docs/README-RU.md">Русский</a>] | [<a href="docs/README-PTBR.md">Português (Brasil)</a>] | [<a href="docs/README-EO.md">Esperanto</a>] | [<a href="docs/README-KR.md">한국어</a>] | [<a href="docs/README-AR.md">العربي</a>] | [<a href="docs/README-VN.md">Tiếng Việt</a>] | [<a href="docs/README-DA.md">Dansk</a>] | [<a href="docs/README-GR.md">Ελληνικά</a>] | [<a href="docs/README-TR.md">Türkçe</a>] | [<a href="docs/README-NO.md">Norsk</a>] | [<a href="docs/README-RO.md">Română</a>]<br>
  <b>We need your help to translate this README, <a href="https://github.com/rustdesk/rustdesk/tree/master/src/lang">RustDesk UI</a> and <a href="https://github.com/rustdesk/doc.rustdesk.com">RustDesk Doc</a> to your native language</b>
</p>

> [!Caution]
> **Misuse Disclaimer:** <br>
> The developers of RustDesk do not condone or support any unethical or illegal use of this software. Misuse, such as unauthorized access, control or invasion of privacy, is strictly against our guidelines. The authors are not responsible for any misuse of the application.

## 自定义 RustDesk 客户端构建、发布和官方同步

本 fork 的生产分支是 `custom-1.4.8` 以及后续 `custom-x.y.z` 分支。不要直接用 GitHub 的 **Sync fork** 按钮同步默认分支，否则可能把官方 `master` 的未发布代码混入自定义编译分支。

### 已内置的客户端配置

当前自定义客户端会内置以下服务端配置：

- ID 服务器：`rustdesk.shbupin.com`
- 中继服务器：`rustdesk.shbupin.com:21117`
- API 服务器：`https://rustdesk.shbupin.com`
- Key：`zOtR7oPxtiXmy6bHXL+as8UGLeXciz+jSVGMEMYEH5s=`
- 官方升级：关闭
- 自定义升级：使用 `https://rustdesk.shbupin.com/update/<platform>.json`

主要自定义代码位置：

- `libs/hbb_common/src/config.rs`：内置 ID / relay / API / key / 默认选项。
- `src/common.rs`：检查自定义升级 JSON。
- `src/updater.rs`：下载并安装自定义升级包。
- `src/ui_interface.rs`：向 Flutter UI 返回客户端版本号。
- `flutter/lib/desktop/pages/desktop_home_page.dart`：允许自定义客户端显示升级卡片。
- `flutter/lib/desktop/widgets/update_progress.dart`：支持直接升级包 URL。
- `.github/workflows/flutter-tag.yml`：编译 Windows / Linux / macOS 桌面客户端，可选同时编译 Android APK 和 unsigned iOS IPA。
- `.github/workflows/publish-custom-release.yml`：生成下载包、升级 JSON 并上传服务器。
- `.github/workflows/sync-upstream-release.yml`：检测官方新 tag，创建新 `custom-*` 分支并触发编译。

### GitHub Secrets

进入 `Settings` -> `Secrets and variables` -> `Actions`，至少配置：

```text
DEPLOY_HOST      ECS 公网 IP
DEPLOY_PORT      22
DEPLOY_USER      root
DEPLOY_KEY       SSH 私钥
DEPLOY_PATH      /www/wwwroot/rustdesk.shbupin.com
```

可选配置：

```text
WORKFLOW_TOKEN   用于跨 workflow 触发构建的 PAT；如果 GITHUB_TOKEN 可以触发 workflow_dispatch，可不填
ANDROID_SIGNING_KEY
ANDROID_ALIAS
ANDROID_KEY_STORE_PASSWORD
ANDROID_KEY_PASSWORD
```

`WORKFLOW_TOKEN` 建议使用 classic PAT，至少具备当前仓库的 `repo` 和 `workflow` 权限。没有这个 token 时，`sync-upstream-release.yml` 会先尝试使用 `GITHUB_TOKEN` 触发 `Flutter Tag Build`。

Android 签名 Secrets 可选。不配置时 workflow 仍会生成 APK，但属于调试/未正式签名包；正式分发 Android 客户端时应配置自己的 keystore 签名信息。

iOS workflow 生成的是 `--no-codesign` 的 unsigned IPA。GitHub Actions 会把未签名的 `Runner.app` 手动打包成 IPA，它只能作为构建产物或后续签名输入。真机安装、TestFlight 或正式分发仍需要 Apple Developer 证书、profile 和签名流程。

### 版本号策略

客户端版本号和升级 JSON 的 `version` 均保持为官方 RustDesk 版本号，例如 `custom-1.4.8` 分支发布 `1.4.8`，`custom-1.4.9` 分支发布 `1.4.9`。

本仓库不再使用 `CUSTOM_VERSION` Secret，也不再发布 `1.4.8-custom.1` 这类版本号。这样做的好处是版本号和官方 release tag 保持一致，后续同步官方新 tag 时不需要手动改版本。

注意：如果只在同一个官方版本分支上重复编译，例如一直在 `custom-1.4.8` 上重编，客户端不会因为同版本 JSON 自动触发升级。需要客户端自动提示升级时，应同步到新的官方 tag，例如 `custom-1.4.9`。如果强行改成 `1.4.8.1` 这类非官方版本号，也能触发升级，但会偏离“跟官方版本保持一致”的策略。

### 手动编译和上传

1. 打开 `Actions`。
2. 运行 `Flutter Tag Build`。
3. Branch 选择当前自定义分支，例如 `custom-1.4.8`。
4. `include_mobile` 默认保持 `true`，会同时编译 Android APK 和 unsigned iOS IPA；如只想编译桌面端可改成 `false`。
5. 构建成功后，`Publish Custom RustDesk Release` 会自动触发。
6. 发布 workflow 会上传：
   - Windows / Linux 客户端安装包到 `/www/wwwroot/rustdesk.shbupin.com/download`
   - 升级 JSON 到 `/www/wwwroot/rustdesk.shbupin.com/update`
   - macOS 升级 JSON 指向 GitHub Release 中的 `.dmg`，不再把 macOS 安装包上传到服务器 `/download`
   - Android APK 和 iOS IPA 会发布到 GitHub Release；发布脚本也会为识别到的移动端包生成对应 JSON

升级 JSON 文件按平台生成，例如：

```text
update/windows-x86_64.json
update/windows-aarch64.json
update/macos-x86_64.json
update/macos-aarch64.json
update/linux-x86_64.json
update/linux-aarch64.json
update/android-aarch64.json
update/android-arm.json
update/android-x86_64.json
update/ios-aarch64.json
```

### 自动同步官方新 tag

`Sync Upstream Release Tag` workflow 支持定时和手动运行。

自动流程：

1. 拉取 `https://github.com/rustdesk/rustdesk` 的 tags。
2. 找到最新的语义化 tag，例如 `1.4.9`。
3. 基于官方 tag 创建 `custom-1.4.9`。
4. 将当前自定义分支中从官方 tag 之后的自定义提交按顺序 cherry-pick 到新分支。
5. 推送 `custom-1.4.9`。
6. 触发 `Flutter Tag Build`，版本号保持为官方版本 `1.4.9`。
7. 构建成功后自动上传到服务器并生成升级 JSON。

手动指定版本：

1. 打开 `Actions` -> `Sync Upstream Release Tag`。
2. `upstream_tag` 填官方 tag，例如 `1.4.9`；留空则自动选最新 tag。
3. `source_custom_branch` 留空时使用当前分支；也可以填 `custom-1.4.8`。
4. `dispatch_build` 保持 `true`。
5. 运行 workflow。

如果冲突只发生在 `libs/hbb_common` 子模块指针，workflow 会自动保留 `source_custom_branch` 中的 hbb_common 指针，确保内置服务器配置不丢失。如果官方源码或其它文件和自定义代码冲突，workflow 仍会在 cherry-pick 阶段失败。此时不要强行覆盖，应该手动解决冲突，确认上面的自定义功能清单没有丢失，再重新运行编译。

### hbb_common 同步说明

本仓库通过子模块使用 `andy--chen/hbb_common.git`。自动同步 RustDesk 新 tag 时，会保留当前自定义提交中的 hbb_common 子模块指针，确保内置服务器配置不丢失。

如果官方新版本要求更新 hbb_common API，可能会出现编译失败。处理方式：

1. 在 `andy--chen/hbb_common` 中基于官方 `rustdesk/hbb_common` 对应提交同步。
2. 重新应用 `libs/hbb_common/src/config.rs` 中的内置 ID / relay / API / key / 升级选项。
3. 推送 hbb_common。
4. 回到本仓库更新 `libs/hbb_common` 子模块指针并提交。
5. 重新运行 `Flutter Tag Build`。

### 分支策略

- `custom-x.y.z`：生产构建分支，默认分支应指向当前使用的 custom 分支。
- `master`：建议保留为参考分支，不建议删除；如果确认不再需要，可在默认分支切到 `custom-*` 后删除。
- 不建议直接把官方 `master` 合并到生产分支。生产客户端优先基于官方 release tag 构建。


Chat with us: [Discord](https://discord.gg/nDceKgxnkV) | [Twitter](https://twitter.com/rustdesk) | [Reddit](https://www.reddit.com/r/rustdesk) | [YouTube](https://www.youtube.com/@rustdesk)

[![RustDesk Server Pro](https://img.shields.io/badge/RustDesk%20Server%20Pro-Advanced%20Features-blue)](https://rustdesk.com/pricing.html)

Yet another remote desktop solution, written in Rust. Works out of the box with no configuration required. You have full control of your data, with no concerns about security. You can use our rendezvous/relay server, [set up your own](https://rustdesk.com/server), or [write your own rendezvous/relay server](https://github.com/rustdesk/rustdesk-server-demo).

![image](https://user-images.githubusercontent.com/71636191/171661982-430285f0-2e12-4b1d-9957-4a58e375304d.png)

RustDesk welcomes contribution from everyone. See [CONTRIBUTING.md](docs/CONTRIBUTING.md) for help getting started.

[**FAQ**](https://github.com/rustdesk/rustdesk/wiki/FAQ)

[**BINARY DOWNLOAD**](https://github.com/rustdesk/rustdesk/releases)

[**NIGHTLY BUILD**](https://github.com/rustdesk/rustdesk/releases/tag/nightly)

[<img src="https://f-droid.org/badge/get-it-on.png"
    alt="Get it on F-Droid"
    height="80">](https://f-droid.org/en/packages/com.carriez.flutter_hbb)
[<img src="https://flathub.org/api/badge?svg&locale=en"
    alt="Get it on Flathub"
    height="80">](https://flathub.org/apps/com.rustdesk.RustDesk)

## Dependencies

Desktop versions use Flutter or Sciter (deprecated) for GUI, this tutorial is for Sciter only, since it is easier and more friendly to start. Check out our [CI](https://github.com/rustdesk/rustdesk/blob/master/.github/workflows/flutter-build.yml) for building Flutter version.

Please download Sciter dynamic library yourself.

[Windows](https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.win/x64/sciter.dll) |
[Linux](https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.lnx/x64/libsciter-gtk.so) |
[macOS](https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.osx/libsciter.dylib)

## Raw Steps to build

- Prepare your Rust development env and C++ build env

- Install [vcpkg](https://github.com/microsoft/vcpkg), and set `VCPKG_ROOT` env variable correctly

  - Windows: vcpkg install libvpx:x64-windows-static libyuv:x64-windows-static opus:x64-windows-static aom:x64-windows-static
  - Linux/macOS: vcpkg install libvpx libyuv opus aom

- run `cargo run`

## [Build](https://rustdesk.com/docs/en/dev/build/)

## How to Build on Linux

### Ubuntu 18 (Debian 10)

```sh
sudo apt install -y zip g++ gcc git curl wget nasm yasm libgtk-3-dev clang libxcb-randr0-dev libxdo-dev \
        libxfixes-dev libxcb-shape0-dev libxcb-xfixes0-dev libasound2-dev libpulse-dev cmake make \
        libclang-dev ninja-build libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libpam0g-dev
```

### openSUSE Tumbleweed

```sh
sudo zypper install gcc-c++ git curl wget nasm yasm gcc gtk3-devel clang libxcb-devel libXfixes-devel cmake alsa-lib-devel gstreamer-devel gstreamer-plugins-base-devel xdotool-devel pam-devel
```

### Fedora 28 (CentOS 8)

```sh
sudo yum -y install gcc-c++ git curl wget nasm yasm gcc gtk3-devel clang libxcb-devel libxdo-devel libXfixes-devel pulseaudio-libs-devel cmake alsa-lib-devel gstreamer1-devel gstreamer1-plugins-base-devel pam-devel
```

### Arch (Manjaro)

```sh
sudo pacman -Syu --needed unzip git cmake gcc curl wget yasm nasm zip make pkg-config clang gtk3 xdotool libxcb libxfixes alsa-lib pipewire
```

### Install vcpkg

```sh
git clone https://github.com/microsoft/vcpkg
cd vcpkg
git checkout 2023.04.15
cd ..
vcpkg/bootstrap-vcpkg.sh
export VCPKG_ROOT=$HOME/vcpkg
vcpkg/vcpkg install libvpx libyuv opus aom
```

### Fix libvpx (For Fedora)

```sh
cd vcpkg/buildtrees/libvpx/src
cd *
./configure
sed -i 's/CFLAGS+=-I/CFLAGS+=-fPIC -I/g' Makefile
sed -i 's/CXXFLAGS+=-I/CXXFLAGS+=-fPIC -I/g' Makefile
make
cp libvpx.a $HOME/vcpkg/installed/x64-linux/lib/
cd
```

### Build

```sh
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
git clone --recurse-submodules https://github.com/rustdesk/rustdesk
cd rustdesk
mkdir -p target/debug
wget https://raw.githubusercontent.com/c-smile/sciter-sdk/master/bin.lnx/x64/libsciter-gtk.so
mv libsciter-gtk.so target/debug
VCPKG_ROOT=$HOME/vcpkg cargo run
```

## How to build with Docker

Begin by cloning the repository and building the Docker container:

```sh
git clone https://github.com/rustdesk/rustdesk
cd rustdesk
git submodule update --init --recursive
docker build -t "rustdesk-builder" .
```

Then, each time you need to build the application, run the following command:

```sh
docker run --rm -it -v $PWD:/home/user/rustdesk -v rustdesk-git-cache:/home/user/.cargo/git -v rustdesk-registry-cache:/home/user/.cargo/registry -e PUID="$(id -u)" -e PGID="$(id -g)" rustdesk-builder
```

Note that the first build may take longer before dependencies are cached, subsequent builds will be faster. Additionally, if you need to specify different arguments to the build command, you may do so at the end of the command in the `<OPTIONAL-ARGS>` position. For instance, if you wanted to build an optimized release version, you would run the command above followed by `--release`. The resulting executable will be available in the target folder on your system, and can be run with:

```sh
target/debug/rustdesk
```

Or, if you're running a release executable:

```sh
target/release/rustdesk
```

Please ensure that you run these commands from the root of the RustDesk repository, or the application may not find the required resources. Also note that other cargo subcommands such as `install` or `run` are not currently supported via this method as they would install or run the program inside the container instead of the host.

## File Structure

- **[libs/hbb_common](https://github.com/rustdesk/rustdesk/tree/master/libs/hbb_common)**: video codec, config, tcp/udp wrapper, protobuf, fs functions for file transfer, and some other utility functions
- **[libs/scrap](https://github.com/rustdesk/rustdesk/tree/master/libs/scrap)**: screen capture
- **[libs/enigo](https://github.com/rustdesk/rustdesk/tree/master/libs/enigo)**: platform specific keyboard/mouse control
- **[libs/clipboard](https://github.com/rustdesk/rustdesk/tree/master/libs/clipboard)**: file copy and paste implementation for Windows, Linux, macOS.
- **[src/ui](https://github.com/rustdesk/rustdesk/tree/master/src/ui)**: obsolete Sciter UI (deprecated)
- **[src/server](https://github.com/rustdesk/rustdesk/tree/master/src/server)**: audio/clipboard/input/video services, and network connections
- **[src/client.rs](https://github.com/rustdesk/rustdesk/tree/master/src/client.rs)**: start a peer connection
- **[src/rendezvous_mediator.rs](https://github.com/rustdesk/rustdesk/tree/master/src/rendezvous_mediator.rs)**: Communicate with [rustdesk-server](https://github.com/rustdesk/rustdesk-server), wait for remote direct (TCP hole punching) or relayed connection
- **[src/platform](https://github.com/rustdesk/rustdesk/tree/master/src/platform)**: platform specific code
- **[flutter](https://github.com/rustdesk/rustdesk/tree/master/flutter)**: Flutter code for desktop and mobile
- **[flutter/web/js](https://github.com/rustdesk/rustdesk/tree/master/flutter/web/v1/js)**: JavaScript for Flutter web client

## Screenshots

![Connection Manager](https://github.com/rustdesk/rustdesk/assets/28412477/db82d4e7-c4bc-4823-8e6f-6af7eadf7651)

![Connected to a Windows PC](https://github.com/rustdesk/rustdesk/assets/28412477/9baa91e9-3362-4d06-aa1a-7518edcbd7ea)

![File Transfer](https://github.com/rustdesk/rustdesk/assets/28412477/39511ad3-aa9a-4f8c-8947-1cce286a46ad)

![TCP Tunneling](https://github.com/rustdesk/rustdesk/assets/28412477/78e8708f-e87e-4570-8373-1360033ea6c5)

