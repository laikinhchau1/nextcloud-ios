# Nextcloud iOS Client - Agent Guide

This document provides essential information for AI coding agents working on the Nextcloud iOS client project.

## Project Overview

The Nextcloud iOS client is the official iOS application for [Nextcloud](https://nextcloud.com), a self-hosted cloud storage and collaboration platform. The app allows users to:

- Access and manage files stored on Nextcloud servers
- Auto-upload photos and videos from the camera roll
- View and edit documents, images, and media
- Share files with others
- Receive push notifications
- Use end-to-end encryption (E2EE)
- Access files through iOS File Provider integration

**Current Codename:** Matheria (represents a pivotal step forward with substantial architectural enhancements)

**License:** GPLv3 with Apple App Store exception (see `COPYING.iOS`)

## Technology Stack

- **Language:** Swift (modern Swift with async/await patterns)
- **UI Framework:** Hybrid UIKit and SwiftUI (UIKit for main interface, SwiftUI for newer components)
- **IDE:** Xcode 26.1+
- **Minimum iOS Version:** As defined in project settings
- **Architecture:** MVVM-like with singleton managers, protocol-oriented programming

### Key Dependencies (Swift Package Manager)

| Package | Purpose |
|---------|---------|
| [NextcloudKit](https://github.com/nextcloud/NextcloudKit) | Official Nextcloud API client library |
| [Realm](https://github.com/realm/realm-swift) | Local database for metadata caching |
| [Firebase](https://github.com/firebase/firebase-ios-sdk) | Crash reporting and analytics |
| [NextcloudKit](https://github.com/nextcloud/NextcloudKit) | Networking and WebDAV operations |
| [SwiftNIO](https://github.com/apple/swift-nio) | Networking framework |
| [OpenSSL](https://github.com/krzyzanowskim/OpenSSL) | Cryptographic operations for E2EE |
| [Queuer](https://github.com/FabrizioBrancati/Queuer) | Background task queue management |
| [EasyTipView](https://github.com/teodorpatras/EasyTipView) | User tips and onboarding |
| [Parchment](https://github.com/rechsteiner/Parchment) | Paging view controller |
| [Mantis](https://github.com/guoyingtao/Mantis) | Image cropping |
| [PopupView](https://github.com/exyte/PopupView) | SwiftUI popup presentations |
| [VLCKit](https://github.com/videolan/vlckit-spm) | Media playback |

## Project Structure

```
nextcloud-ios/
├── iOSClient/                    # Main application source code
│   ├── AppDelegate.swift         # Application lifecycle
│   ├── NCGlobal.swift            # Global constants and configuration
│   ├── Account/                  # Account management
│   ├── Activity/                 # Activity feed UI
│   ├── Assistant/                # AI Assistant integration (SwiftUI)
│   ├── AudioRecorder/            # Voice memo recording
│   ├── BrowserWeb/               # In-app web browser
│   ├── Data/                     # Database layer (Realm)
│   │   └── NCManageDatabase+*.swift  # Database extensions by feature
│   ├── DeepLink/                 # URL scheme handling
│   ├── Extensions/               # Swift extensions
│   ├── Favorites/                # Favorite files UI
│   ├── Files/                    # File browser UI
│   ├── GUI/                      # Shared UI components
│   ├── Login/                    # Authentication flows
│   ├── Main/                     # Main navigation controllers
│   ├── Media/                    # Photo/video gallery
│   ├── Menu/                     # Context menus
│   ├── More/                     # Settings and more options
│   ├── Networking/               # Network operations
│   │   ├── NCNetworking+*.swift  # Networking extensions
│   │   └── E2EE/                 # End-to-end encryption
│   └── Utility/                  # Helper utilities
├── Brand/                        # Brand configuration and assets
│   ├── NCBrand.swift             # Brand options and theming
│   ├── *.entitlements            # App entitlements
│   └── *.plist                   # Info plists
├── Share/                        # Share Extension (iOS Share Sheet)
├── File\ Provider\ Extension/    # iOS Files app integration
├── File\ Provider\ Extension\ UI/ # File Provider UI
├── Notification\ Service\ Extension/ # Push notification handling
├── Widget/                       # iOS Home Screen widgets
├── WidgetDashboardIntentHandler/ # Widget intent handling
├── Tests/                        # Test suites
│   ├── NextcloudUnitTests/       # Unit tests (minimal)
│   ├── NextcloudIntegrationTests/ # Integration tests
│   └── NextcloudUITests/         # UI automation tests
├── ExternalResources/            # External resource handlers
└── Nextcloud.xcodeproj/          # Xcode project
```

## Build and Development Setup

### Prerequisites

1. **Xcode 26.1+** (as specified in README)
2. **macOS** with development tools
3. **GoogleService-Info.plist** - Required for Firebase integration
   - For development, use the mock version:
   ```bash
   wget "https://raw.githubusercontent.com/firebase/quickstart-ios/master/mock-GoogleService-Info.plist" -O GoogleService-Info.plist
   ```

### Build Commands

```bash
# Build the main target
xcodebuild build-for-testing \
  -scheme "Nextcloud" \
  -destination "platform=iOS Simulator,name=iPhone 16,OS=18.5" \
  -derivedDataPath "DerivedData"

# Build additional targets
xcodebuild build -project Nextcloud.xcodeproj -scheme "Share" -destination "platform=iOS Simulator,name=iPhone 16"
xcodebuild build -project Nextcloud.xcodeproj -scheme "File Provider Extension" -destination "platform=iOS Simulator,name=iPhone 16"
xcodebuild build -project Nextcloud.xcodeproj -scheme "Widget" -destination "platform=iOS Simulator,name=iPhone 16"
xcodebuild build -project Nextcloud.xcodeproj -scheme "Notification Service Extension" -destination "platform=iOS Simulator,name=iPhone 16"
```

## Testing

### Test Organization

Tests are located in `Tests/` directory:

- **Unit Tests** (`NextcloudUnitTests/`): Minimal unit tests (currently placeholders)
- **Integration Tests** (`NextcloudIntegrationTests/`): Server-dependent tests
- **UI Tests** (`NextcloudUITests/`): End-to-end automation tests

### Test Server Setup

For integration and UI tests, a Nextcloud server is required:

```bash
# Use the convenience script (review contents first)
./Tests/Server.sh
```

This script:
1. Starts a Docker container with Nextcloud server (stable30 branch)
2. Exposes server on `http://localhost:8080`
3. Enables required apps (assistant, testing, files_downloadlimit)

### Test Configuration

Configure test credentials in `Tests/TestConstants.swift`:

```swift
static let server = "http://localhost:8080"
static let username = "admin"
static let password = "admin"
```

### Running Tests

```bash
# Run tests (requires server running for integration/UI tests)
xcodebuild test-without-building \
  -xctestrun $(find . -type f -name "*.xctestrun") \
  -destination "platform=iOS Simulator,name=iPhone 16,OS=18.5"
```

**Note:** UI tests assume the device/simulator is set to **US English** locale.

**Important:** If a test exclusively tests NextcloudKit functionality, add it to the [NextcloudKit](https://github.com/nextcloud/NextcloudKit) project instead.

## Code Style Guidelines

### SwiftLint

The project uses [SwiftLint](https://github.com/realm/SwiftLint) for code style enforcement.

Configuration in `.swiftlint.yml`:
- Line length: warning at 1000, error at 5000
- Function body length: warning at 400
- Type body length: warning at 2000, error at 2500
- File length: warning at 2000, error at 2500
- Disabled rules include: `cyclomatic_complexity`, `nesting`, `type_name`

Excluded directories:
- `Tests/`
- `Brand/NCBrand.swift`
- `iOSClient/NCGlobal.swift`
- `iOSClient/Utility/NCLivePhoto.swift`

### Coding Conventions

1. **File Headers:** Use SPDX license headers:
   ```swift
   // SPDX-FileCopyrightText: Nextcloud GmbH
   // SPDX-FileCopyrightText: <year> <author>
   // SPDX-License-Identifier: GPL-3.0-or-later
   ```

2. **Copyright:** Contributors should add copyright for substantial changes:
   ```swift
   // @copyright Copyright (c) <year>, <your name> (<your email>)
   ```

3. **Naming:**
   - Classes: `NC` prefix for Nextcloud classes (e.g., `NCNetworking`)
   - Database extensions: `NCManageDatabase+<Feature>.swift`
   - Networking extensions: `NCNetworking+<Feature>.swift`

4. **Global Constants:** Define in `NCGlobal.swift`:
   - Notification names: `notificationCenter<Name>`
   - Error codes: `error<Name>`
   - Status codes: `metadataStatus<Name>`

5. **Async/Await:** Modern code uses Swift async/await patterns:
   ```swift
   Task {
       await someAsyncOperation()
   }
   ```

## Key Architecture Components

### Database Layer (`iOSClient/Data/`)

Uses Realm for local metadata caching. Organized as extensions:
- `NCManageDatabase+Account.swift` - Account management
- `NCManageDatabase+Metadata.swift` - File metadata
- `NCManageDatabase+E2EE.swift` - End-to-end encryption
- `NCManageDatabase+Share.swift` - Sharing data

### Networking Layer (`iOSClient/Networking/`)

Built on NextcloudKit with extensions:
- `NCNetworking+WebDAV.swift` - WebDAV operations
- `NCNetworking+Upload.swift` - File uploads
- `NCNetworking+Download.swift` - File downloads
- `NCNetworking+E2EE/` - Encryption operations

### Brand Configuration (`Brand/NCBrand.swift`)

Central configuration for:
- Brand name and colors
- Capabilities groups
- Feature toggles
- AppConfig/MDM support

### Global Constants (`iOSClient/NCGlobal.swift`)

Singleton pattern for:
- Error codes
- Notification names
- Layout constants
- Version checks

## Security Considerations

### End-to-End Encryption (E2EE)

- Implementation in `iOSClient/Networking/E2EE/`
- Uses OpenSSL for cryptographic operations
- Compatible versions: "1.1", "1.2", "2.0", "2.1"
- Passphrase test defined in `NCGlobal.shared.e2eePassphraseTest`

### Certificate Handling

- Certificates stored in `Library/Application Support/Certificates/`
- SSL certificate validation with user prompt for untrusted certs
- View certificate details in `NCViewCertificateDetails`

### Push Notifications

- Push proxy: `https://push-notifications.nextcloud.com`
- Device tokens stored securely
- Notification encryption support

### App Security

- Passcode lock support with `TOPasscodeViewController`
- Biometric authentication (Face ID/Touch ID)
- Privacy screen option
- Certificate pinning support

## CI/CD

GitHub Actions workflows in `.github/workflows/`:

1. **xcode.yml** - Main build and test workflow
   - Runs on macOS-15 with Xcode 26.0.1
   - Builds for iPhone 16 Simulator
   - Tests are currently disabled (see workflow comments)

2. **lint.yml** - SwiftLint validation
   - Runs on Ubuntu with SwiftLint action

3. **additional-targets.yml** - Build share extension, widgets, etc.

## Contribution Requirements

### DCO Signoff

All commits must include a Signed-off-by line:
```bash
git commit -s -m "Your commit message"
```

Format:
```
My Commit message

Signed-off-by: Random Contributor <random@contributor.dev>
```

### Pull Request Process

1. Fork the repository
2. Create feature branch from `develop`
3. Make changes with appropriate tests
4. Ensure SwiftLint passes
5. Submit PR to `develop` branch
6. Link related issues

## Common Development Tasks

### Adding a New Feature

1. Identify the appropriate module in `iOSClient/`
2. Create/modify files following naming conventions
3. Add database methods in `NCManageDatabase+<Feature>.swift` if needed
4. Add networking methods in `NCNetworking+<Feature>.swift` if needed
5. Add tests in appropriate `Tests/` subdirectory

### Working with the Database

```swift
// Example: Fetching metadata
let metadatas = await NCManageDatabase.shared.getMetadatasAsync(...)

// Example: Adding metadata
await NCManageDatabase.shared.addMetadataAsync(metadata)
```

### Working with Networking

```swift
// Example: Upload
let error = await NCNetworking.shared.uploadFileInBackground(metadata: metadata)

// Example: WebDAV operations
let result = await NCNetworking.shared.readFileOrFolder(...)
```

## External Resources

- **Documentation:** https://docs.nextcloud.com/
- **Forum:** https://help.nextcloud.com/c/clients/ios
- **Issue Tracker:** https://github.com/nextcloud/ios/issues
- **TestFlight:** https://testflight.apple.com/join/RXEJbWj9
- **Security Reports:** https://hackerone.com/nextcloud

## Troubleshooting

### Common Issues

1. **Build fails with Firebase error**: Ensure `GoogleService-Info.plist` exists
2. **SwiftLint errors**: Check `.swiftlint.yml` exclusions
3. **Tests fail with server error**: Ensure Docker server is running via `Tests/Server.sh`
4. **Realm migration errors**: Clean build folder and delete app from simulator

### Debug Logging

Enable debug logging in `AppDelegate.swift`:
```swift
NextcloudKit.configureLogger(logLevel: .debug)
```

Log tags defined in `NCGlobal.swift`:
- `BGT` - Background tasks
- `E2EE` - Encryption
- `SYNC` - Synchronization
- `DB` - Database operations
