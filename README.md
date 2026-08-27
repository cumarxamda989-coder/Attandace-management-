name: Build Android APK

on:
  push:
    branches: [ main, master ]
  workflow_dispatch:

jobs:
  build:
    name: Build APK
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install Dependencies
        run: |
          npm install --legacy-peer-deps || npm install

      - name: Build Web App
        run: |
          npm run build

      - name: Setup Java JDK
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: '17'

      - name: Setup Capacitor & Android Platform
        run: |
          npm install @capacitor/core @capacitor/cli @capacitor/android
          npx cap init "Attendance Work" "com.attendance.work" --web-dir "dist" || true
          npx cap add android || true
          npx cap sync android

      - name: Build Android APK
        run: |
          cd android
          chmod +x gradlew
          ./gradlew assembleDebug --no-daemon

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: attendance-work-app
          path: android/app/build/outputs/apk/debug/app-debug.apk
          retention-days: 30
