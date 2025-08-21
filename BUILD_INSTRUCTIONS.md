# Build Instructions for Screen Lock Android App

## Prerequisites

### System Requirements
- Ubuntu 22.04 or later (or compatible Linux distribution)
- At least 8GB RAM and 20GB free disk space
- Internet connection for downloading dependencies

### Required Software
1. **Flutter SDK 3.35.1 or higher**
2. **Android SDK with API level 33**
3. **Java 17 JDK**
4. **Git** (for version control)

## Step-by-Step Setup

### 1. Install Flutter
```bash
# Install snapd if not already installed
sudo apt update
sudo apt install -y snapd

# Install Flutter via snap
sudo snap install flutter --classic

# Add Flutter to PATH
export PATH="$PATH:/snap/bin"
echo 'export PATH="$PATH:/snap/bin"' >> ~/.bashrc
source ~/.bashrc
```

### 2. Install Java 17
```bash
sudo apt update
sudo apt install -y openjdk-17-jdk

# Set JAVA_HOME
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
```

### 3. Install Android SDK
```bash
# Download Android command line tools
cd ~
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip

# Extract and set up
mkdir -p ~/android-sdk/cmdline-tools
unzip commandlinetools-linux-11076708_latest.zip -d ~/android-sdk/cmdline-tools
mv ~/android-sdk/cmdline-tools/cmdline-tools ~/android-sdk/cmdline-tools/latest

# Set environment variables
export ANDROID_HOME=~/android-sdk
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
echo 'export ANDROID_HOME=~/android-sdk' >> ~/.bashrc
echo 'export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin' >> ~/.bashrc

# Configure Flutter to use Android SDK
flutter config --android-sdk ~/android-sdk

# Accept licenses and install required packages
yes | sdkmanager --licenses
sdkmanager "platform-tools" "platforms;android-33" "build-tools;33.0.0"
```

### 4. Verify Setup
```bash
flutter doctor
```
You should see checkmarks for Flutter and Android toolchain.

## Building the App

### 1. Get Dependencies
```bash
cd screen_lock_app
flutter pub get
```

### 2. Generate Assets
```bash
# Generate app icons
flutter pub run flutter_launcher_icons

# Generate splash screen
flutter pub run flutter_native_splash:create
```

### 3. Build Release APK
```bash
# Set Java home for the build
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64

# Build the release APK
flutter build apk --release
```

### 4. Locate the APK
The built APK will be located at:
```
build/app/outputs/flutter-apk/app-release.apk
```

## Build Variants

### Debug Build
For development and testing:
```bash
flutter build apk --debug
```

### Profile Build
For performance testing:
```bash
flutter build apk --profile
```

### Split APKs by Architecture
To create separate APKs for different architectures:
```bash
flutter build apk --split-per-abi
```

## Troubleshooting

### Common Build Issues

#### 1. Java Version Conflicts
If you encounter Java version errors:
```bash
# Check current Java version
java -version

# Ensure Java 17 is being used
sudo update-alternatives --config java
```

#### 2. Android SDK Issues
If Android SDK is not found:
```bash
# Verify ANDROID_HOME is set
echo $ANDROID_HOME

# Re-configure Flutter
flutter config --android-sdk $ANDROID_HOME
```

#### 3. Gradle Build Failures
If Gradle build fails:
```bash
# Clean the project
flutter clean

# Get dependencies again
flutter pub get

# Try building again
flutter build apk --release
```

#### 4. NDK Issues
If you encounter NDK-related errors, install the NDK:
```bash
sdkmanager "ndk;25.1.8937393"
```

### Memory Issues
If the build fails due to memory issues:
```bash
# Increase Gradle memory
export GRADLE_OPTS="-Xmx4g -XX:MaxPermSize=512m"
```

## Optimization Tips

### 1. Reduce APK Size
```bash
# Build with obfuscation
flutter build apk --release --obfuscate --split-debug-info=debug-info/

# Build App Bundle (recommended for Play Store)
flutter build appbundle --release
```

### 2. Enable R8 Shrinking
Add to `android/app/build.gradle`:
```gradle
android {
    buildTypes {
        release {
            shrinkResources true
            minifyEnabled true
        }
    }
}
```

### 3. Optimize Images
Ensure all images in `assets/` are optimized for mobile use.

## Testing the Build

### 1. Install on Device
```bash
# Enable USB debugging on your Android device
# Connect device via USB
adb install build/app/outputs/flutter-apk/app-release.apk
```

### 2. Test Key Features
- Test all three lock types (number, pattern, alphanumeric)
- Verify biometric authentication (if device supports it)
- Test wrong password attempts and camera capture
- Check animations and UI responsiveness

## Deployment Preparation

### 1. App Signing
For production deployment, you'll need to sign the APK:
```bash
# Generate a signing key
keytool -genkey -v -keystore ~/key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias key

# Configure signing in android/app/build.gradle
```

### 2. Version Management
Update version in `pubspec.yaml`:
```yaml
version: 1.0.0+1
```

### 3. Final Testing
- Test on multiple devices and Android versions
- Verify all permissions work correctly
- Test in different lighting conditions (for camera feature)
- Verify biometric authentication on supported devices

## Build Automation

### Using GitHub Actions
Create `.github/workflows/build.yml`:
```yaml
name: Build APK
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: actions/setup-java@v1
      with:
        java-version: '17'
    - uses: subosito/flutter-action@v1
    - run: flutter pub get
    - run: flutter build apk --release
```

## Performance Monitoring

### Build Time Optimization
```bash
# Use build cache
flutter build apk --release --build-shared-library

# Parallel builds
flutter build apk --release --dart-define=flutter.inspector.structuredErrors=true
```

### APK Analysis
```bash
# Analyze APK size
flutter build apk --analyze-size

# Generate size report
flutter build apk --release --analyze-size --target-platform android-arm64
```

---

**Note**: Build times may vary depending on system specifications and network speed. The first build typically takes longer due to dependency downloads and Gradle setup.

