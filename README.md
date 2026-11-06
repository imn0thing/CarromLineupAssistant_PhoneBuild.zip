# CarromLineupAssistant_PhoneBuild.zip

name: Build Carrom APK

on:
  workflow_dispatch:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "17"

      - name: Install Android SDK
        run: |
          SDKMANAGER=$(find "$ANDROID_HOME/cmdline-tools" -name sdkmanager -type f | head -n 1)
          yes | "$SDKMANAGER" --licenses || true
          "$SDKMANAGER" "platform-tools" "platforms;android-35" "build-tools;35.0.0"

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: "8.9"

      - name: Extract project if needed
        run: |
          if [ -f settings.gradle.kts ]; then
            echo "Project already extracted"
          else
            mkdir -p extracted
            unzip -q CarromLineupAssistant_PhoneBuild.zip -d extracted
            cd extracted
          fi

      - name: Build APK
        run: |
          if [ -f settings.gradle.kts ]; then
            gradle assembleDebug
          else
            cd extracted
            gradle -p "$(find . -name settings.gradle.kts -printf '%h\n' | head -n 1)" assembleDebug
          fi

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: Carrom-Lineup-APK
          path: app/build/outputs/apk/debug/app-debug.apk
          
