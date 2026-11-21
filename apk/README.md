# Release APKs

The signed catalog pins the APK by name, size and SHA-256. Upload the exact file the publisher script hashed; a different build is rejected by the app before install.

Current catalog entry:

- file: `nestdroid-1.6.30.apk`
- size: `8844195` bytes
- SHA-256: `bf52388b9142d7199aaf8bd7bf17bd27ed5188d230a25e8064725e9f2aa91100`

This file is not in the repository yet. Until it is uploaded here, the channel advertises 1.6.30 and the app reports an HTTP 404 when it tries to download it. Upload it from the project's `build/dist/` directory; the app verifies the digest above before installing.
