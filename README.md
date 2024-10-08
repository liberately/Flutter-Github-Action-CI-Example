# Flutter-Github-Action-CI-Example

This example demonstrates how to use GitHub Actions to build packages for Windows, Linux, macOS, Android, iOS, and Web platforms simultaneously using Flutter.

## Dependencies

- [subosito/flutter-action](https://github.com/subosito/flutter-action)
- [actions/checkout](https://github.com/actions/checkout)
- [actions/upload-artifact](https://github.com/actions/upload-artifact)
- [actions/setup-java](https://github.com/actions/setup-java)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)

## Prerequisites

- Ensure your Flutter project is correctly configured.
- Enable GitHub Actions in your GitHub repository.
- Set `Workflow permissions` to `Read and write permissions`. Located in `Settings` > `Code and automation` > `Action` > `General`.
- Configure necessary certificates and keys, especially for iOS and Android.

## Workflow Configuration

Create a `.github/workflows/build.yml` file in the root directory of your Flutter project and add the following content:

```yaml
name: Flutter Build

on:
  workflow_dispatch:
  push:
    tags:
      - '*'


jobs:
  web:
    runs-on: ubuntu-latest
    steps:
      - name: Set up repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build web
        run: flutter build web --release

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: web-build
          path: build/web/

      - name: Create ZIP of compiled files
        run: tar -czvf web.tar.gz -C build/web .

      - name: Publish release
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/tags/')
        with:
          files: web.tar.gz
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          draft: false
          prerelease: false

  linux:
    runs-on: ubuntu-latest
    steps:
      - name: Set up repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2

      - name: Install dependencies
        run: |
          sudo apt-get update -y
          sudo apt-get install -y ninja-build libgtk-3-dev

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build for Linux
        run: flutter build linux --release

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: linux-build
          path: build/linux/x64/release/bundle/

      - name: Create tar.gz archive
        run: tar -czvf linux-x86_64.tar.gz -C build/linux/x64/release/bundle .

      - name: Publish release
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/tags/')
        with:
          files: linux-x86_64.tar.gz
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          draft: false
          prerelease: false

  android:
    runs-on: ubuntu-latest
    steps:
      - name: Set up repository
        uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Set up Flutter
        uses: subosito/flutter-action@v2

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build APK
        run: flutter build apk --release

      - name: Build split APKs
        run: flutter build apk --split-per-abi

      - name: Build App Bundle
        run: flutter build appbundle --release

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: android-build
          path: build/app/outputs/

      - name: Rename APKs and AABs
        run: |
          mv build/app/outputs/flutter-apk/app-release.apk build/app/outputs/flutter-apk/app-universal-release.apk
          mv build/app/outputs/bundle/release/app-release.aab build/app/outputs/bundle/release/app-universal-release.aab

      - name: Publish release
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/tags/')
        with:
          files: |
            build/app/outputs/flutter-apk/app-universal-release.apk
            build/app/outputs/flutter-apk/app-armeabi-v7a-release.apk
            build/app/outputs/flutter-apk/app-arm64-v8a-release.apk
            build/app/outputs/flutter-apk/app-x86_64-release.apk
            build/app/outputs/bundle/release/app-universal-release.aab
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          draft: false
          prerelease: false

  windows:
    runs-on: windows-latest
    steps:
      - name: Set up repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build for Windows
        run: flutter build windows --release

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: windows-build
          path: build/windows/

      - name: Zip compiled files
        run: Compress-Archive -Path build/windows/x64/runner/Release/* -DestinationPath windows-x86_64.zip

      - name: Publish release
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/tags/')
        with:
          files: windows-x86_64.zip
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          draft: false
          prerelease: false

  macos:
    runs-on: macos-latest
    steps:
      - name: Set up repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build for macOS
        run: flutter build macos --release

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: macos-build
          path: build/macos/

      - name: Zip compiled files
        run: zip -r macos-universal-release.zip build/macos/Build/Products/Release/app.app

      - name: Publish release
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/tags/')
        with:
          files: macos-universal-release.zip
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          draft: false
          prerelease: false

  ios:
    runs-on: macos-latest
    steps:
      - name: Set up repository
        uses: actions/checkout@v4

      - name: Set up Flutter
        uses: subosito/flutter-action@v2

      - name: Install dependencies
        run: flutter pub get

      - name: Run tests
        run: flutter test

      - name: Build for iOS
        run: flutter build ios --release --no-codesign

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: ios-build
          path: build/ios/

      - name: Zip compiled files
        run: zip -r ios-universal-release.zip build/ios/iphoneos/Runner.app

      - name: Publish release
        uses: softprops/action-gh-release@v2
        if: startsWith(github.ref, 'refs/tags/')
        with:
          files: ios-universal-release.zip
          tag_name: ${{ github.ref_name }}
          name: Release ${{ github.ref_name }}
          draft: false
          prerelease: false
```

## Build Steps

1. **Checkout Code**: Check out code from the GitHub repository.
2. **Set up Flutter**: Install the specified version of Flutter.
3. **Install Dependencies**: Run `flutter pub get` to install project dependencies.
4. **Build Platforms**: Build release packages for Android, iOS, Web, Windows, Linux, and macOS in order.

## Usage Instructions

1. Add the above `build.yml` file to your GitHub repository.
2. Whenever you push code with tags, GitHub Actions will automatically trigger the build process.
3. After

a successful build, you can view the build logs and generated binaries in the Actions panel.

![](./doc/42d6b845959177a9334d882ce474b541_MD5.jpeg)

## Contributions

Contributions are welcome; please submit issues or pull requests!