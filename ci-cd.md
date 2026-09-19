# CI/CD: Auto-build APK with GitHub Actions

Automate your build so every push gives you a fresh APK without touching Android Studio.

## Workflow

Create `.github/workflows/build.yml`:

```yaml
name: Build APK

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 17

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v3

      - name: Build debug APK
        run: ./gradlew assembleDebug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: app-debug
          path: app/build/outputs/apk/debug/app-debug.apk
```

## Signed Release Build

For release builds, store your keystore as a GitHub secret:

1. Base64 encode your keystore: `base64 -i keystore.jks -o keystore.txt`
2. Add as repository secret: `KEYSTORE_BASE64`
3. Add secrets: `KEY_ALIAS`, `KEY_PASSWORD`, `STORE_PASSWORD`

```yaml
      - name: Decode keystore
        run: echo ${{ secrets.KEYSTORE_BASE64 }} | base64 -d > app/keystore.jks

      - name: Build signed APK
        run: ./gradlew assembleRelease
        env:
          SIGNING_KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          SIGNING_KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
          SIGNING_STORE_PASSWORD: ${{ secrets.STORE_PASSWORD }}
```

## Tips

- Use `workflow_dispatch` so you can trigger builds manually from GitHub UI
- Cache Gradle dependencies to speed up builds
- Upload APK as artifact so you can download directly from Actions tab
- Consider adding a version bump step based on commit count

## Why This Matters

When you're iterating fast — changing animations, tweaking reactions, adding features — you don't want to plug in your phone and hit build every time. Push code, wait 2 minutes, download APK, install. Done.
