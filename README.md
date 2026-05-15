# Namma-Raste Reporter - Development SOP
## **Version 2.3** - Updated for Android Studio Panda | 2025.3.1

---

## 📋 PROJECT METADATA

**Project Name:** Namma-Raste Reporter  
**Platform:** Android (Mobile-only)  
**Primary Language:** Kotlin  
**Architecture:** MVVM + Clean Architecture  
**Minimum SDK:** API 24 (Android 7.0 Nougat)  
**Target SDK:** API 36 (Android 16)  
**Package Name:** `com.nammaraste.reporter`

**Build Configuration:**
- **Android Studio:** Panda | 2025.3.1
- **Android Gradle Plugin (AGP):** 9.1.0
- **Kotlin:** 2.1.0
- **Gradle:** 9.0
- **Java:** 21 (JDK 21)

---

## 🎯 CORE PHILOSOPHY

### The "3-Click Rule"
Every feature must adhere to: **Open App → Capture/Select Photo → Submit Report** in ≤ 3 user taps.

#### 3-Click Path - Live Camera Capture
1. **Click 1:** Tap Shutter Button (captures photo + GPS + timestamp)
2. **Click 2:** Select Issue Type & Severity
3. **Click 3:** Tap Submit

#### 3-Click Path - Gallery Upload
1. **Click 1:** Tap Gallery Icon
2. **Click 2:** Select Photo from System Picker (auto-extracts GPS + timestamp from EXIF)
3. **Click 3:** Tap Submit on auto-populated Classification Screen

---

## 🏗️ TECHNICAL STACK

### Language & Conventions
- **Language:** Kotlin 2.1.0 (100% Kotlin, zero Java files)
- **Kotlin Compiler:** K2 compiler (default in Kotlin 2.1.0)
- **Naming Conventions:**
  - Classes: `PascalCase` (e.g., `ReportViewModel`)
  - Variables/Functions: `camelCase` (e.g., `capturePhoto()`)
  - Constants: `SCREAMING_SNAKE_CASE` (e.g., `MAX_PHOTO_SIZE`)
  - **Linting:** All code must pass `ktlint` with 0 errors

### Architecture Pattern
```
MVVM + Clean Architecture
├── UI Layer (Fragments/Activities - XML Views)
├── ViewModel (StateFlow preferred over LiveData)
├── Repository (Data abstraction)
├── Data Sources
    ├── Room Database (Local)
    └── Firebase Firestore (Optional Cloud Sync)
```

---

## 📊 DATABASE SCHEMA

### Room Database Entities

#### **Entity: Report**
```kotlin
@Entity(tableName = "reports")
data class Report(
    @PrimaryKey val ticketId: String,        // NR-YYYYMMDD-XXXXX
    val userId: String,                       // Foreign key to User
    val photoPath: String,                    // Local file path
    val issueType: String,                    // Pothole | Broken Streetlight | Damaged Road Sign | Other
    val severity: String,                     // Low | Medium | High
    val latitude: Double,                     // GPS coordinate
    val longitude: Double,                    // GPS coordinate
    val timestamp: String,                    // ISO 8601 UTC
    val status: String,                       // Submitted | Under Review | Work Assigned | Resolved
    val isFromGallery: Boolean                // True if uploaded from gallery, false if live camera
)
```

#### **Entity: User (SIMPLIFIED - No Profile Photo, No DigiLocker)**
```kotlin
@Entity(
    tableName = "users",
    indices = [Index(value = ["mobileNumber"], unique = true)]
)
data class User(
    @PrimaryKey val userId: String,           // Auto-generated UUID
    val name: String,                          // User's full name
    val mobileNumber: String,                  // 10-digit mobile (unique)
    val passwordHash: String                   // bcrypt hash, never plaintext
)
```

**Simplifications Applied:**
- ❌ Removed: Profile photo/avatar field
- ❌ Removed: Email field (only mobile number for identification)
- ❌ Removed: DigiLocker integration fields
- ✅ Kept: Essential fields only (name, mobile, password)

### Ticket ID Format
```
Pattern: NR-YYYYMMDD-XXXXX
Example: NR-20250624-00042

Where:
- NR = Namma-Raste prefix
- YYYYMMDD = Date of submission
- XXXXX = 5-digit auto-increment or UUID fragment
```

---

## 📦 DEPENDENCIES (UPDATED FOR AGP 9.1.0 & API 24+)

### Project-level build.gradle.kts
```kotlin
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    alias(libs.plugins.android.application) apply false
    alias(libs.plugins.kotlin.android) apply false
    alias(libs.plugins.kotlin.compose) apply false
    alias(libs.plugins.hilt) apply false
    alias(libs.plugins.navigation.safeargs) apply false
    alias(libs.plugins.google.services) apply false
    alias(libs.plugins.ksp) apply false
}
```

### gradle/libs.versions.toml (Version Catalog - Modern Approach)
```toml
[versions]
# Build Tools
agp = "9.1.0"
kotlin = "2.1.0"
ksp = "2.1.0-1.0.29"

# AndroidX Core
core-ktx = "1.15.0"
appcompat = "1.7.0"
material = "1.12.0"
constraintlayout = "2.2.0"
activity-ktx = "1.9.3"
fragment-ktx = "1.8.5"

# Lifecycle
lifecycle = "2.8.7"

# Navigation
navigation = "2.8.5"

# CameraX
camerax = "1.4.1"

# Location
play-services-location = "21.3.0"

# ExifInterface
exifinterface = "1.3.7"

# Room
room = "2.6.1"

# Hilt
hilt = "2.54"

# Firebase
firebase-bom = "33.8.0"

# Security
security-crypto = "1.1.0-alpha06"

# BCrypt
jbcrypt = "0.4"

# Coil
coil = "3.0.4"

# Coroutines
coroutines = "1.10.1"

# DataStore
datastore = "1.1.1"

# Testing
junit = "4.13.2"
androidx-test-ext-junit = "1.2.1"
espresso-core = "3.6.1"
arch-core-testing = "2.2.0"

[libraries]
# AndroidX Core
androidx-core-ktx = { group = "androidx.core", name = "core-ktx", version.ref = "core-ktx" }
androidx-appcompat = { group = "androidx.appcompat", name = "appcompat", version.ref = "appcompat" }
material = { group = "com.google.android.material", name = "material", version.ref = "material" }
androidx-constraintlayout = { group = "androidx.constraintlayout", name = "constraintlayout", version.ref = "constraintlayout" }
androidx-activity-ktx = { group = "androidx.activity", name = "activity-ktx", version.ref = "activity-ktx" }
androidx-fragment-ktx = { group = "androidx.fragment", name = "fragment-ktx", version.ref = "fragment-ktx" }

# Lifecycle
androidx-lifecycle-viewmodel-ktx = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-ktx", version.ref = "lifecycle" }
androidx-lifecycle-livedata-ktx = { group = "androidx.lifecycle", name = "lifecycle-livedata-ktx", version.ref = "lifecycle" }
androidx-lifecycle-runtime-ktx = { group = "androidx.lifecycle", name = "lifecycle-runtime-ktx", version.ref = "lifecycle" }
androidx-lifecycle-viewmodel-savedstate = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-savedstate", version.ref = "lifecycle" }

# Navigation
androidx-navigation-fragment-ktx = { group = "androidx.navigation", name = "navigation-fragment-ktx", version.ref = "navigation" }
androidx-navigation-ui-ktx = { group = "androidx.navigation", name = "navigation-ui-ktx", version.ref = "navigation" }

# CameraX
androidx-camera-core = { group = "androidx.camera", name = "camera-core", version.ref = "camerax" }
androidx-camera-camera2 = { group = "androidx.camera", name = "camera-camera2", version.ref = "camerax" }
androidx-camera-lifecycle = { group = "androidx.camera", name = "camera-lifecycle", version.ref = "camerax" }
androidx-camera-view = { group = "androidx.camera", name = "camera-view", version.ref = "camerax" }
androidx-camera-extensions = { group = "androidx.camera", name = "camera-extensions", version.ref = "camerax" }

# Location
play-services-location = { group = "com.google.android.gms", name = "play-services-location", version.ref = "play-services-location" }

# ExifInterface
androidx-exifinterface = { group = "androidx.exifinterface", name = "exifinterface", version.ref = "exifinterface" }

# Room
androidx-room-runtime = { group = "androidx.room", name = "room-runtime", version.ref = "room" }
androidx-room-ktx = { group = "androidx.room", name = "room-ktx", version.ref = "room" }
androidx-room-compiler = { group = "androidx.room", name = "room-compiler", version.ref = "room" }
androidx-room-testing = { group = "androidx.room", name = "room-testing", version.ref = "room" }

# Hilt
hilt-android = { group = "com.google.dagger", name = "hilt-android", version.ref = "hilt" }
hilt-compiler = { group = "com.google.dagger", name = "hilt-compiler", version.ref = "hilt" }
hilt-android-testing = { group = "com.google.dagger", name = "hilt-android-testing", version.ref = "hilt" }

# Firebase
firebase-bom = { group = "com.google.firebase", name = "firebase-bom", version.ref = "firebase-bom" }
firebase-auth-ktx = { group = "com.google.firebase", name = "firebase-auth-ktx" }
firebase-firestore-ktx = { group = "com.google.firebase", name = "firebase-firestore-ktx" }
firebase-storage-ktx = { group = "com.google.firebase", name = "firebase-storage-ktx" }

# Security
androidx-security-crypto = { group = "androidx.security", name = "security-crypto", version.ref = "security-crypto" }

# BCrypt
jbcrypt = { group = "org.mindrot", name = "jbcrypt", version.ref = "jbcrypt" }

# Coil
coil = { group = "io.coil-kt.coil3", name = "coil", version.ref = "coil" }
coil-network-okhttp = { group = "io.coil-kt.coil3", name = "coil-network-okhttp", version.ref = "coil" }

# Coroutines
kotlinx-coroutines-android = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-android", version.ref = "coroutines" }
kotlinx-coroutines-core = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-core", version.ref = "coroutines" }
kotlinx-coroutines-test = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-test", version.ref = "coroutines" }

# DataStore
androidx-datastore-preferences = { group = "androidx.datastore", name = "datastore-preferences", version.ref = "datastore" }

# Testing
junit = { group = "junit", name = "junit", version.ref = "junit" }
androidx-test-ext-junit = { group = "androidx.test.ext", name = "junit", version.ref = "androidx-test-ext-junit" }
androidx-espresso-core = { group = "androidx.test.espresso", name = "espresso-core", version.ref = "espresso-core" }
androidx-arch-core-testing = { group = "androidx.arch.core", name = "core-testing", version.ref = "arch-core-testing" }

[plugins]
android-application = { id = "com.android.application", version.ref = "agp" }
kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
kotlin-compose = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
navigation-safeargs = { id = "androidx.navigation.safeargs.kotlin", version.ref = "navigation" }
google-services = { id = "com.google.gms.google-services", version = "4.4.2" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

### App-level build.gradle.kts
```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.hilt)
    alias(libs.plugins.navigation.safeargs)
    alias(libs.plugins.google.services)
    alias(libs.plugins.ksp)
}

android {
    namespace = "com.nammaraste.reporter"
    compileSdk = 36

    defaultConfig {
        applicationId = "com.nammaraste.reporter"
        minSdk = 24  // Android 7.0 Nougat - API 24
        targetSdk = 36
        versionCode = 1
        versionName = "1.0.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        
        vectorDrawables {
            useSupportLibrary = true
        }
        
        // Room schema export location
        ksp {
            arg("room.schemaLocation", "$projectDir/schemas")
            arg("room.incremental", "true")
            arg("room.generateKotlin", "true")
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            signingConfig = signingConfigs.getByName("debug")
        }
        debug {
            isMinifyEnabled = false
            isDebuggable = true
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-DEBUG"
        }
    }
    
    compileOptions {
        // Support for Java 21 features
        sourceCompatibility = JavaVersion.VERSION_21
        targetCompatibility = JavaVersion.VERSION_21
        
        // Enable desugaring for API 24+ compatibility
        isCoreLibraryDesugaringEnabled = true
    }
    
    kotlinOptions {
        jvmTarget = "21"
        
        // Kotlin 2.1.0 optimizations
        freeCompilerArgs += listOf(
            "-opt-in=kotlin.RequiresOptIn",
            "-opt-in=kotlinx.coroutines.ExperimentalCoroutinesApi",
            "-opt-in=kotlinx.coroutines.FlowPreview",
            "-Xcontext-receivers"
        )
    }
    
    buildFeatures {
        viewBinding = true
        dataBinding = true
        buildConfig = true
    }
    
    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
            excludes += "/META-INF/gradle/incremental.annotation.processors"
        }
    }
    
    // Lint options
    lint {
        checkReleaseBuilds = true
        abortOnError = false
        warningsAsErrors = false
    }
}

dependencies {
    // Desugaring for Java 21 features on API 24+
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.1.4")
    
    // AndroidX Core
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.appcompat)
    implementation(libs.material)
    implementation(libs.androidx.constraintlayout)
    implementation(libs.androidx.activity.ktx)
    implementation(libs.androidx.fragment.ktx)
    
    // Lifecycle
    implementation(libs.androidx.lifecycle.viewmodel.ktx)
    implementation(libs.androidx.lifecycle.livedata.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.lifecycle.viewmodel.savedstate)
    
    // Navigation
    implementation(libs.androidx.navigation.fragment.ktx)
    implementation(libs.androidx.navigation.ui.ktx)
    
    // CameraX
    implementation(libs.androidx.camera.core)
    implementation(libs.androidx.camera.camera2)
    implementation(libs.androidx.camera.lifecycle)
    implementation(libs.androidx.camera.view)
    implementation(libs.androidx.camera.extensions)
    
    // Location Services
    implementation(libs.play.services.location)
    
    // ExifInterface
    implementation(libs.androidx.exifinterface)
    
    // Room Database (using KSP)
    implementation(libs.androidx.room.runtime)
    implementation(libs.androidx.room.ktx)
    ksp(libs.androidx.room.compiler)
    
    // Firebase
    implementation(platform(libs.firebase.bom))
    implementation(libs.firebase.auth.ktx)
    implementation(libs.firebase.firestore.ktx)
    implementation(libs.firebase.storage.ktx)
    
    // Hilt (using KSP)
    implementation(libs.hilt.android)
    ksp(libs.hilt.compiler)
    
    // Security
    implementation(libs.androidx.security.crypto)
    
    // BCrypt
    implementation(libs.jbcrypt)
    
    // Coil Image Loading
    implementation(libs.coil)
    implementation(libs.coil.network.okhttp)
    
    // Coroutines
    implementation(libs.kotlinx.coroutines.android)
    implementation(libs.kotlinx.coroutines.core)
    
    // DataStore (Optional - Modern replacement for SharedPreferences)
    implementation(libs.androidx.datastore.preferences)
    
    // Testing
    testImplementation(libs.junit)
    testImplementation(libs.kotlinx.coroutines.test)
    testImplementation(libs.androidx.arch.core.testing)
    
    androidTestImplementation(libs.androidx.test.ext.junit)
    androidTestImplementation(libs.androidx.espresso.core)
    androidTestImplementation(libs.androidx.room.testing)
    androidTestImplementation(libs.hilt.android.testing)
    kspAndroidTest(libs.hilt.compiler)
}
```

### gradle.properties (IMPORTANT UPDATES)
```properties
# Project-wide Gradle settings
org.gradle.jvmargs=-Xmx4096m -Dfile.encoding=UTF-8 -XX:+UseParallelGC
org.gradle.parallel=true
org.gradle.caching=true
org.gradle.configuration-cache=true
org.gradle.daemon=true

# AndroidX package structure
android.useAndroidX=true

# Kotlin code style
kotlin.code.style=official

# Non-transitive R class
android.nonTransitiveRClass=true

# Enable Kotlin K2 compiler (default in Kotlin 2.1.0)
kotlin.experimental.tryK2=true

# KSP incremental processing
ksp.incremental=true
ksp.incremental.log=true

# Enable build features
android.defaults.buildfeatures.buildconfig=true
android.defaults.buildfeatures.aidl=false
android.defaults.buildfeatures.renderscript=false
android.defaults.buildfeatures.resvalues=false
android.defaults.buildfeatures.shaders=false

# Enable R8 full mode for better optimization
android.enableR8.fullMode=true

# Disable unused features for faster builds
android.enableJetifier=false

# AGP 9.1.0 optimizations
android.experimental.enableScreenshotTest=false
```

### gradle-wrapper.properties
```properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-9.0-bin.zip
networkTimeout=10000
validateDistributionUrl=true
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

---

## 📱 ANDROID MANIFEST (UPDATED FOR API 24+)

### AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- Camera permission -->
    <uses-permission android:name="android.permission.CAMERA" />
    
    <!-- Location permissions -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    
    <!-- Media location permission (Android 10+) -->
    <uses-permission android:name="android.permission.ACCESS_MEDIA_LOCATION" />
    
    <!-- Internet for Firebase -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    
    <!-- Photo picker and storage (Android version specific) -->
    <!-- Android 13+ (API 33+) -->
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
    
    <!-- Android 10-12 (API 29-32) -->
    <uses-permission 
        android:name="android.permission.READ_EXTERNAL_STORAGE"
        android:maxSdkVersion="32" />
    
    <!-- Android 7-9 (API 24-28) - Our minimum SDK -->
    <uses-permission 
        android:name="android.permission.WRITE_EXTERNAL_STORAGE"
        android:maxSdkVersion="28"
        tools:ignore="ScopedStorage" />

    <!-- Feature declarations -->
    <uses-feature 
        android:name="android.hardware.camera" 
        android:required="true" />
    <uses-feature 
        android:name="android.hardware.camera.autofocus"
        android:required="false" />
    <uses-feature 
        android:name="android.hardware.location.gps" 
        android:required="false" />

    <application
        android:name=".NammaRasteApplication"
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.NammaRaste"
        android:enableOnBackInvokedCallback="true"
        android:networkSecurityConfig="@xml/network_security_config"
        tools:targetApi="tiramisu">
        
        <activity
            android:name=".ui.MainActivity"
            android:exported="true"
            android:screenOrientation="portrait"
            android:windowSoftInputMode="adjustResize"
            android:configChanges="orientation|screenSize|keyboardHidden">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        
        <!-- File Provider for camera photos -->
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
        
        <!-- Firebase Analytics (optional) -->
        <meta-data
            android:name="firebase_analytics_collection_enabled"
            android:value="false" />
    </application>

</manifest>
```

### res/xml/network_security_config.xml (NEW - Required for API 24+)
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!-- Allow cleartext traffic for localhost (debug only) -->
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">localhost</domain>
    </domain-config>
    
    <!-- Base configuration for all connections -->
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

### res/xml/file_paths.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- External files directory (no permission needed on API 19+) -->
    <external-files-path 
        name="my_images" 
        path="Pictures/" />
    
    <!-- Cache directory for temporary files -->
    <cache-path 
        name="my_cache_images" 
        path="/" />
    
    <!-- Internal storage (rare use case) -->
    <files-path 
        name="my_internal" 
        path="/" />
</paths>
```

### res/xml/data_extraction_rules.xml (Android 12+)
```xml
<?xml version="1.0" encoding="utf-8"?>
<data-extraction-rules>
    <cloud-backup>
        <include domain="database" path="." />
        <exclude domain="sharedpref" path="namma_raste_secure_prefs_master_key.xml" />
    </cloud-backup>
    
    <device-transfer>
        <include domain="database" path="." />
        <exclude domain="sharedpref" path="namma_raste_secure_prefs_master_key.xml" />
    </device-transfer>
</data-extraction-rules>
```

### res/xml/backup_rules.xml (Legacy - API < 31)
```xml
<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <include domain="database" path="." />
    <exclude domain="sharedpref" path="namma_raste_secure_prefs_master_key.xml" />
</full-backup-content>
```

---

## 🏗️ PROJECT STRUCTURE

```
app/
├── src/
│   ├── main/
│   │   ├── kotlin/com/nammaraste/reporter/
│   │   │   ├── NammaRasteApplication.kt
│   │   │   ├── data/
│   │   │   │   ├── local/
│   │   │   │   │   ├── dao/
│   │   │   │   │   │   ├── ReportDao.kt
│   │   │   │   │   │   └── UserDao.kt
│   │   │   │   │   ├── entities/
│   │   │   │   │   │   ├── Report.kt
│   │   │   │   │   │   └── User.kt (SIMPLIFIED)
│   │   │   │   │   └── AppDatabase.kt
│   │   │   │   ├── remote/
│   │   │   │   │   ├── FirebaseAuthService.kt
│   │   │   │   │   └── FirestoreService.kt
│   │   │   │   ├── repository/
│   │   │   │   │   ├── ReportRepository.kt
│   │   │   │   │   └── UserRepository.kt
│   │   │   │   └── models/
│   │   │   │       └── PhotoMetadata.kt
│   │   │   ├── di/
│   │   │   │   ├── AppModule.kt
│   │   │   │   ├── DatabaseModule.kt
│   │   │   │   └── NetworkModule.kt
│   │   │   ├── ui/
│   │   │   │   ├── MainActivity.kt
│   │   │   │   ├── auth/
│   │   │   │   │   ├── LoginFragment.kt (SIMPLIFIED)
│   │   │   │   │   └── LoginViewModel.kt
│   │   │   │   ├── camera/
│   │   │   │   │   ├── CameraFragment.kt
│   │   │   │   │   └── CameraViewModel.kt
│   │   │   │   ├── classification/
│   │   │   │   │   ├── ClassificationFragment.kt
│   │   │   │   │   └── ClassificationViewModel.kt
│   │   │   │   ├── success/
│   │   │   │   │   └── SuccessFragment.kt (NO SHARING)
│   │   │   │   ├── history/
│   │   │   │   │   ├── HistoryFragment.kt
│   │   │   │   │   ├── HistoryAdapter.kt
│   │   │   │   │   └── HistoryViewModel.kt
│   │   │   │   └── tracker/
│   │   │   │       ├── TrackerFragment.kt
│   │   │   │       └── TrackerViewModel.kt
│   │   │   └── utils/
│   │   │       ├── Constants.kt
│   │   │       ├── TicketIdGenerator.kt
│   │   │       ├── PermissionUtils.kt
│   │   │       ├── MetadataHelper.kt
│   │   │       └── CompatibilityUtils.kt (NEW - API 24+ compatibility helpers)
│   │   ├── res/
│   │   │   ├── layout/
│   │   │   │   ├── activity_main.xml
│   │   │   │   ├── fragment_login.xml (SIMPLIFIED UI)
│   │   │   │   ├── fragment_camera.xml
│   │   │   │   ├── fragment_classification.xml
│   │   │   │   ├── fragment_success.xml (NO SHARE BUTTONS)
│   │   │   │   ├── fragment_history.xml
│   │   │   │   ├── fragment_tracker.xml
│   │   │   │   └── item_report_card.xml
│   │   │   ├── navigation/
│   │   │   │   └── nav_graph.xml
│   │   │   ├── values/
│   │   │   │   ├── strings.xml
│   │   │   │   ├── colors.xml
│   │   │   │   └── themes.xml
│   │   │   ├── drawable/
│   │   │   │   ├── ic_camera.xml
│   │   │   │   ├── ic_gallery.xml
│   │   │   │   ├── ic_pothole.xml
│   │   │   │   ├── ic_streetlight.xml
│   │   │   │   └── ic_road_sign.xml
│   │   │   └── xml/
│   │   │       ├── file_paths.xml
│   │   │       ├── data_extraction_rules.xml
│   │   │       ├── backup_rules.xml
│   │   │       └── network_security_config.xml (NEW)
│   │   └── AndroidManifest.xml
│   ├── test/
│   │   └── kotlin/com/nammaraste/reporter/
│   │       ├── MetadataHelperTest.kt
│   │       ├── TicketIdGeneratorTest.kt
│   │       └── ReportRepositoryTest.kt
│   └── androidTest/
│       └── kotlin/com/nammaraste/reporter/
│           ├── DatabaseTest.kt
│           └── NavigationTest.kt
├── schemas/  (Room database schemas)
├── build.gradle.kts
└── proguard-rules.pro
```

---

## 🔐 SECURITY SPECIFICATIONS

### Password Security
```kotlin
// NEVER store plaintext passwords
// Use bcrypt with minimum 10 rounds
import org.mindrot.jbcrypt.BCrypt

fun hashPassword(plainPassword: String): String {
    return BCrypt.hashpw(plainPassword, BCrypt.gensalt(10))
}

fun verifyPassword(plainPassword: String, hashedPassword: String): Boolean {
    return BCrypt.checkpw(plainPassword, hashedPassword)
}
```

### Session Management (EncryptedSharedPreferences - API 24+ Compatible)
```kotlin
import androidx.security.crypto.EncryptedSharedPreferences
import androidx.security.crypto.MasterKey

// Create MasterKey (works on API 23+)
fun createMasterKey(context: Context): MasterKey {
    return MasterKey.Builder(context)
        .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
        .build()
}

// Create EncryptedSharedPreferences
fun getEncryptedPrefs(context: Context): SharedPreferences {
    val masterKey = createMasterKey(context)
    
    return EncryptedSharedPreferences.create(
        context,
        "namma_raste_secure_prefs",
        masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
}

// Usage
val encryptedPrefs = getEncryptedPrefs(context)
encryptedPrefs.edit()
    .putString("current_user_id", userId)
    .apply()

val userId = encryptedPrefs.getString("current_user_id", null)
```

---

## 📡 PERMISSIONS MANAGEMENT (API 24+ COMPATIBLE)

### Required Permissions by Android Version

| Permission | API 24-28 | API 29-32 | API 33+ | Purpose |
|------------|-----------|-----------|---------|---------|
| `CAMERA` | ✅ | ✅ | ✅ | Live photo capture |
| `ACCESS_FINE_LOCATION` | ✅ | ✅ | ✅ | GPS tagging |
| `ACCESS_COARSE_LOCATION` | ✅ | ✅ | ✅ | Fallback location |
| `ACCESS_MEDIA_LOCATION` | ❌ | ✅ | ✅ | EXIF GPS (API 29+) |
| `WRITE_EXTERNAL_STORAGE` | ✅ | ❌ | ❌ | Storage (API 24-28) |
| `READ_EXTERNAL_STORAGE` | ✅ | ✅ | ❌ | Gallery (API 24-32) |
| `READ_MEDIA_IMAGES` | ❌ | ❌ | ✅ | Gallery (API 33+) |

### Modern Permission Request (API 24+ Compatible)
```kotlin
import android.Manifest
import android.content.pm.PackageManager
import android.os.Build
import androidx.activity.result.contract.ActivityResultContracts
import androidx.core.content.ContextCompat

class CameraFragment : Fragment() {

    private val permissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestMultiplePermissions()
    ) { permissions ->
        handlePermissionsResult(permissions)
    }
    
    fun requestAllPermissions() {
        val permissions = mutableListOf<String>()
        
        // Camera permission (all versions)
        permissions.add(Manifest.permission.CAMERA)
        
        // Location permissions (all versions)
        permissions.add(Manifest.permission.ACCESS_FINE_LOCATION)
        permissions.add(Manifest.permission.ACCESS_COARSE_LOCATION)
        
        // Media location permission (API 29+)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
            permissions.add(Manifest.permission.ACCESS_MEDIA_LOCATION)
        }
        
        // Storage permissions (version-specific)
        when {
            Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU -> {
                // Android 13+ (API 33+)
                permissions.add(Manifest.permission.READ_MEDIA_IMAGES)
            }
            Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q -> {
                // Android 10-12 (API 29-32)
                permissions.add(Manifest.permission.READ_EXTERNAL_STORAGE)
            }
            else -> {
                // Android 7-9 (API 24-28)
                permissions.add(Manifest.permission.READ_EXTERNAL_STORAGE)
                permissions.add(Manifest.permission.WRITE_EXTERNAL_STORAGE)
            }
        }
        
        permissionLauncher.launch(permissions.toTypedArray())
    }
    
    private fun handlePermissionsResult(permissions: Map<String, Boolean>) {
        val cameraGranted = permissions[Manifest.permission.CAMERA] == true
        val locationGranted = permissions[Manifest.permission.ACCESS_FINE_LOCATION] == true
        
        val storageGranted = when {
            Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU -> {
                permissions[Manifest.permission.READ_MEDIA_IMAGES] == true
            }
            else -> {
                permissions[Manifest.permission.READ_EXTERNAL_STORAGE] == true
            }
        }
        
        when {
            cameraGranted && locationGranted && storageGranted -> {
                initializeCameraAndGallery()
            }
            !cameraGranted -> {
                showPermissionDeniedDialog("Camera permission is required")
            }
            !locationGranted -> {
                showPermissionDeniedDialog("Location permission is required for GPS tagging")
            }
            !storageGranted -> {
                showPermissionDeniedDialog("Storage permission is required to upload photos")
            }
        }
    }
    
    fun hasAllPermissions(): Boolean {
        val context = requireContext()
        
        val cameraGranted = ContextCompat.checkSelfPermission(
            context, Manifest.permission.CAMERA
        ) == PackageManager.PERMISSION_GRANTED
        
        val locationGranted = ContextCompat.checkSelfPermission(
            context, Manifest.permission.ACCESS_FINE_LOCATION
        ) == PackageManager.PERMISSION_GRANTED
        
        val storageGranted = when {
            Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU -> {
                ContextCompat.checkSelfPermission(
                    context, Manifest.permission.READ_MEDIA_IMAGES
                ) == PackageManager.PERMISSION_GRANTED
            }
            else -> {
                ContextCompat.checkSelfPermission(
                    context, Manifest.permission.READ_EXTERNAL_STORAGE
                ) == PackageManager.PERMISSION_GRANTED
            }
        }
        
        return cameraGranted && locationGranted && storageGranted
    }
}
```

### CompatibilityUtils.kt (NEW - API 24+ Helper)
```kotlin
package com.nammaraste.reporter.utils

import android.Manifest
import android.content.Context
import android.content.pm.PackageManager
import android.os.Build
import androidx.core.content.ContextCompat

object CompatibilityUtils {
    
    /**
     * Get storage permission based on Android version
     */
    fun getStoragePermission(): String {
        return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            Manifest.permission.READ_MEDIA_IMAGES
        } else {
            Manifest.permission.READ_EXTERNAL_STORAGE
        }
    }
    
    /**
     * Check if storage permission is granted
     */
    fun hasStoragePermission(context: Context): Boolean {
        return ContextCompat.checkSelfPermission(
            context,
            getStoragePermission()
        ) == PackageManager.PERMISSION_GRANTED
    }
    
    /**
     * Check if media location permission is needed and granted
     */
    fun hasMediaLocationPermission(context: Context): Boolean {
        return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
            ContextCompat.checkSelfPermission(
                context,
                Manifest.permission.ACCESS_MEDIA_LOCATION
            ) == PackageManager.PERMISSION_GRANTED
        } else {
            true // Not needed on API < 29
        }
    }
    
    /**
     * Check if app uses scoped storage
     */
    fun usesScopedStorage(): Boolean {
        return Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q
    }
    
    /**
     * Check if app uses Photo Picker
     */
    fun usesPhotoPicker(): Boolean {
        return Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU
    }
}
```

---

## ⚡ PERFORMANCE BENCHMARKS

| Metric | Target | Verification Method |
|--------|--------|---------------------|
| Camera preview FPS | ≥30 fps | Manual QA on API 24 device |
| Report save time | <2 seconds | Automated instrumentation test |
| GPS accuracy (live capture) | Within 50 meters | Field test at known coordinates |
| **EXIF metadata extraction** | **<500 ms** | **Coroutine execution time measurement** |
| Room DB query time | <200 ms | Benchmark test with 10,000 records |
| App cold start (API 24) | <4 seconds | Profiler measurement on low-end device |
| App cold start (API 33+) | <3 seconds | Profiler measurement on modern device |
| Battery drain per report | <1% | Battery historian analysis |
| Gallery photo copy time | <1 second | Measured for 5MB image |
| Build time (clean with KSP) | <3 minutes | Gradle build with version catalog |

---

## 🎨 UI/UX DESIGN SPECIFICATIONS

### Material Design 3 Guidelines
- **Color Scheme:** Dynamic color (Material You) with fallback for API 24-30
- **Typography:** Roboto font family
- **Components:**
  - MaterialButton (filled, outlined, text variants)
  - MaterialCardView (elevated, outlined)
  - MaterialChip for badges and selections
  - ExtendedFloatingActionButton for primary actions

### Responsive Design
- Support screen sizes: 4.5" to 7" phones
- Support tablets: 7" to 10" (responsive layouts)
- Orientation: Portrait-only (locked in manifest)
- Minimum touch target: 48dp × 48dp (accessibility)
- Font scaling: Support up to 200% (accessibility)

### Accessibility (WCAG 2.1 Level AA)
- All interactive elements have `contentDescription`
- TalkBack fully functional
- Minimum contrast ratio: 4.5:1
- Font scaling support up to 200%
- Keyboard navigation support

### Color Coding Standards
```kotlin
// Severity colors (Material 3 compatible)
object SeverityColors {
    const val LOW = "#4CAF50"      // Green
    const val MEDIUM = "#FF9800"   // Orange
    const val HIGH = "#F44336"     // Red
}

// Status colors
object StatusColors {
    const val SUBMITTED = "#9E9E9E"       // Gray
    const val UNDER_REVIEW = "#2196F3"    // Blue
    const val WORK_ASSIGNED = "#FF9800"   // Orange
    const val RESOLVED = "#4CAF50"        // Green
}
```

---

## 🧪 TESTING STRATEGY (API 24+ COMPATIBLE)

### Unit Tests
```kotlin
import kotlinx.coroutines.test.runTest
import org.junit.Test
import org.junit.Assert.*

class MetadataHelperTest {
    
    @Test
    fun testExtractExifMetadataWithGps() = runTest {
        // Test implementation
    }
    
    @Test
    fun testExtractExifMetadataWithoutGps() = runTest {
        // Test implementation with fallback
    }
    
    @Test
    fun testPerformanceUnder500ms() = runTest {
        val startTime = System.currentTimeMillis()
        metadataHelper.extractExifMetadata(testPhotoUri)
        val duration = System.currentTimeMillis() - startTime
        assertTrue("EXIF extraction took ${duration}ms", duration < 500)
    }
}

class TicketIdGeneratorTest {
    
    @Test
    fun testUniqueIdGeneration() {
        val ids = List(50) { TicketIdGenerator.generate() }
        assertEquals(50, ids.distinct().size)
    }
    
    @Test
    fun testFormatValidation() {
        val ticketId = TicketIdGenerator.generate()
        val regex = "^NR-\\d{8}-[A-Z0-9]{5}$".toRegex()
        assertTrue(regex.matches(ticketId))
    }
}
```

### Instrumented Tests (API 24+ Compatible)
```kotlin
import androidx.test.ext.junit.runners.AndroidJUnit4
import androidx.test.platform.app.InstrumentationRegistry
import org.junit.Test
import org.junit.runner.RunWith
import org.junit.Assert.*

@RunWith(AndroidJUnit4::class)
class DatabaseTest {
    
    @Test
    fun testRoomDatabaseOnApi24() {
        val context = InstrumentationRegistry.getInstrumentation().targetContext
        val db = Room.inMemoryDatabaseBuilder(context, AppDatabase::class.java)
            .allowMainThreadQueries()
            .build()
        
        // Test database operations
        assertNotNull(db)
        db.close()
    }
}
```

### Multi-API Level Testing
```kotlin
// Test on different API levels
@SdkSuppress(minSdkVersion = 24, maxSdkVersion = 28)
@Test
fun testLegacyStoragePermission() {
    // Test for API 24-28
}

@SdkSuppress(minSdkVersion = 29, maxSdkVersion = 32)
@Test
fun testScopedStoragePermission() {
    // Test for API 29-32
}

@SdkSuppress(minSdkVersion = 33)
@Test
fun testPhotoPickerPermission() {
    // Test for API 33+
}
```

---

## 🎯 KEY SIMPLIFICATIONS SUMMARY

### 1. Authentication (LoginFragment)

**REMOVED:**
- ❌ Profile photo upload (ImageView + button)
- ❌ Email field
- ❌ DigiLocker integration button/SDK
- ❌ Social login buttons (Google, Facebook, etc.)
- ❌ OTP verification

**KEPT:**
- ✅ Name field (visible in registration mode only)
- ✅ Mobile number field (10 digits, unique identifier)
- ✅ Password field (minimum 8 characters, bcrypt hashed)
- ✅ Login/Register toggle
- ✅ Session persistence via EncryptedSharedPreferences

### 2. User Entity

**REMOVED:**
- ❌ `profilePhotoPath: String?`
- ❌ `email: String?`
- ❌ `digiLockerVerified: Boolean`
- ❌ `aadhaarNumber: String?`
- ❌ `verificationDocuments: List<String>`

**KEPT:**
- ✅ `userId: String` (UUID, primary key)
- ✅ `name: String` (full name)
- ✅ `mobileNumber: String` (unique, 10 digits)
- ✅ `passwordHash: String` (bcrypt)

### 3. Success Screen (SuccessFragment)

**REMOVED:**
- ❌ "Share via WhatsApp" button
- ❌ Generic "Share" button (Intent.ACTION_SEND)
- ❌ Social media sharing options
- ❌ QR code generation for ticket ID

**KEPT:**
- ✅ Ticket ID display (large, monospace, selectable)
- ✅ "Copy Ticket ID" button (clipboard only)
- ✅ "View My Reports" button (navigate to HistoryFragment)
- ✅ "Submit Another Report" button (navigate to CameraFragment)

---

## 📋 FUNCTIONAL REQUIREMENTS

### FR-01: User Registration (SIMPLIFIED)
**Input Fields:**
- Full Name (minimum 2 characters)
- Mobile Number (exactly 10 digits, numeric only)
- Password (minimum 8 characters)

**Validation:**
- Name: Non-empty, no special characters except spaces
- Mobile: 10 digits, starts with 6-9 (Indian format), unique in database
- Password: Minimum 8 characters, at least 1 letter and 1 number

**Process:**
1. Validate inputs client-side
2. Hash password using bcrypt (10 rounds)
3. Generate UUID for userId
4. Insert User entity into Room DB
5. Store userId in EncryptedSharedPreferences
6. Navigate to CameraFragment

### FR-02: User Login (SIMPLIFIED)
**Input Fields:**
- Mobile Number (10 digits)
- Password

**Process:**
1. Validate mobile number format
2. Query User by mobileNumber from Room DB
3. Verify password using BCrypt.checkpw()
4. Store userId in EncryptedSharedPreferences
5. Navigate to CameraFragment

### FR-03: Camera Capture (CameraX 1.4.1 - API 24+ Compatible)
**Features:**
- Full-screen CameraX preview with Camera2 interop
- Shutter button (FloatingActionButton)
- Gallery button (IconButton)
- Single photo per report
- Auto-focus support (if hardware available)

**On Capture:**
1. Save photo to app-specific cache directory
2. Read GPS via FusedLocationProviderClient
3. Record timestamp (ISO 8601 UTC)
4. Navigate to ClassificationFragment with data

### FR-04: Gallery Upload with EXIF Extraction (API 24+ Compatible)
**Features:**
- Use Photo Picker on API 33+ (ActivityResultContracts.PickVisualMedia)
- Use legacy Intent.ACTION_PICK on API 24-32
- Request appropriate storage permissions based on Android version
- Extract GPS from EXIF metadata (androidx.exifinterface)
- Fallback to device GPS if EXIF missing

**On Selection:**
1. Open InputStream from URI
2. Create ExifInterface instance
3. Extract GPS (lat/lon) and timestamp
4. If GPS null: use FusedLocationProviderClient, set isGpsEstimated=true
5. Navigate to ClassificationFragment with data

### FR-05: Issue Classification
**Input Fields:**
- Issue Type: Pothole | Broken Streetlight | Damaged Road Sign | Other
- Severity: Low | Medium | High

**Display:**
- Photo thumbnail (loaded with Coil 3.0.4)
- GPS source badge (from photo / estimated)
- Coordinates (lat, lon to 4 decimal places)
- Timestamp (formatted for locale)

**On Submit:**
1. Generate unique Ticket ID
2. Copy photo to permanent app-specific storage
3. Create Report entity with all data
4. Insert into Room DB (using KSP-generated code)
5. Navigate to SuccessFragment

### FR-06: Success Display (NO SHARING)
**Display:**
- Animated checkmark (vector drawable)
- Ticket ID (large, monospace, selectable)
- Success message

**Actions:**
- Copy Ticket ID to clipboard
- View My Reports (HistoryFragment)
- Submit Another Report (CameraFragment)

### FR-07: Report History
**Display:**
- RecyclerView of user's reports
- Each card shows:
  - Ticket ID (monospace)
  - Issue type + icon
  - Submission date (relative time)
  - Status badge (color-coded)
  - Photo thumbnail (Coil 3.0.4)
  - Source badge (camera/gallery icon)

**Features:**
- Search by Ticket ID (debounced 300ms)
- Pull-to-refresh (optional Firebase sync)
- Empty state with "Submit First Report" button
- Efficient scrolling with DiffUtil

### FR-08: Status Tracker
**Input:**
- Ticket ID (validated format: NR-YYYYMMDD-XXXXX)

**Display:**
- Photo thumbnail
- Source badge (camera/gallery)
- Issue type + severity
- GPS coordinates
- Timestamp (formatted)
- Timeline view (4 stages)

**Timeline Stages:**
1. Submitted (always completed, green)
2. Under Review (conditional, blue)
3. Work Assigned (conditional, orange)
4. Resolved (conditional, green)

---

## 🔄 DATA FLOW

### Registration Flow (API 24+ Compatible)
```
User Input (Name, Mobile, Password)
    ↓
Client-side Validation
    ↓
BCrypt Hash Password (10 rounds)
    ↓
Generate UUID (java.util.UUID)
    ↓
Insert User → Room DB (via KSP-generated DAO)
    ↓
Store userId → EncryptedSharedPreferences (AES256_GCM)
    ↓
Navigate to CameraFragment (SafeArgs)
```

### Live Camera Capture Flow (API 24+ Compatible)
```
User Taps Shutter
    ↓
CameraX ImageCapture.takePicture()
    ↓
Save to Cache Directory (app-specific, no permission needed)
    ↓
FusedLocationProviderClient.lastLocation (requires runtime permission)
    ↓
Generate ISO 8601 Timestamp (UTC)
    ↓
PhotoMetadata (uri, lat, lon, timestamp, isFromGallery=false, isGpsEstimated=false)
    ↓
Navigate to ClassificationFragment (SafeArgs)
```

### Gallery Upload Flow (API 24-36 Compatible)
```
User Taps Gallery Button
    ↓
Check Android Version:
    IF API 33+ (Tiramisu): Launch Photo Picker (PickVisualMedia)
    ELSE: Launch Intent.ACTION_PICK
    ↓
Request Appropriate Permission:
    IF API 33+: READ_MEDIA_IMAGES
    ELSE IF API 29-32: READ_EXTERNAL_STORAGE
    ELSE (API 24-28): READ_EXTERNAL_STORAGE + WRITE_EXTERNAL_STORAGE
    ↓
User Selects Photo → Receive URI
    ↓
ExifInterface (1.3.7) → Extract GPS & Timestamp
    ↓
IF GPS Exists:
    PhotoMetadata (uri, exifLat, exifLon, exifTimestamp, isFromGallery=true, isGpsEstimated=false)
ELSE:
    FusedLocationProviderClient → Get Device GPS
    PhotoMetadata (uri, deviceLat, deviceLon, currentTimestamp, isFromGallery=true, isGpsEstimated=true)
    ↓
Navigate to ClassificationFragment (SafeArgs)
```

### Report Submission Flow
```
User Selects Issue Type + Severity
    ↓
User Taps Submit (3rd click)
    ↓
Generate Ticket ID (NR-YYYYMMDD-XXXXX)
    ↓
Copy Photo from Cache/Gallery to Permanent Storage
    Target: context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)
    Filename: ${ticketId}.jpg
    ↓
Create Report Entity
    ↓
Insert Report → Room DB (Coroutine on Dispatchers.IO)
    ↓
[Optional] Sync to Firestore (Firebase BOM 33.8.0)
    ↓
Navigate to SuccessFragment with Ticket ID
    ↓
Clear Navigation Backstack (prevent re-submission)
```

---

## 🗂️ FILE PATHS & STORAGE (API 24+ SCOPED STORAGE)

### Photo Storage Strategy by Android Version

**API 24-28 (Android 7-9):**
```kotlin
// Legacy storage with WRITE_EXTERNAL_STORAGE permission
val picturesDir = if (Build.VERSION.SDK_INT < Build.VERSION_CODES.Q) {
    File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES), "NammaRaste")
} else {
    context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)
}
```

**API 29+ (Android 10+):**
```kotlin
// Scoped storage (no permission needed for app-specific directory)
val picturesDir = context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)
val permanentFile = File(picturesDir, "${ticketId}.jpg")
```

### Complete Storage Implementation
```kotlin
object StorageUtils {
    
    /**
     * Get app-specific pictures directory (works on all API levels)
     */
    fun getPicturesDirectory(context: Context): File {
        val dir = context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)
        if (dir != null && !dir.exists()) {
            dir.mkdirs()
        }
        return dir ?: context.filesDir
    }
    
    /**
     * Get cache directory for temporary photos
     */
    fun getCacheDirectory(context: Context): File {
        val cacheDir = File(context.cacheDir, "camera_cache")
        if (!cacheDir.exists()) {
            cacheDir.mkdirs()
        }
        return cacheDir
    }
    
    /**
     * Copy photo from temp to permanent storage
     */
    suspend fun copyPhotoToPermanentStorage(
        context: Context,
        sourceUri: Uri,
        ticketId: String
    ): String = withContext(Dispatchers.IO) {
        val picturesDir = getPicturesDirectory(context)
        val permanentFile = File(picturesDir, "$ticketId.jpg")
        
        context.contentResolver.openInputStream(sourceUri)?.use { input ->
            FileOutputStream(permanentFile).use { output ->
                input.copyTo(output)
            }
        }
        
        permanentFile.absolutePath
    }
    
    /**
     * Clean up temporary cache files
     */
    fun cleanupCacheFiles(context: Context) {
        val cacheDir = getCacheDirectory(context)
        cacheDir.listFiles()?.forEach { file ->
            if (file.lastModified() < System.currentTimeMillis() - 86400000) { // 24 hours
                file.delete()
            }
        }
    }
}
```

### Database Location (All API Levels)
```
Internal Storage: /data/data/com.nammaraste.reporter/databases/namma_raste_db
```

### Encrypted Preferences (All API Levels)
```
Internal Storage: /data/data/com.nammaraste.reporter/shared_prefs/namma_raste_secure_prefs_master_key.xml
(Encrypted using AES256_GCM via Jetpack Security - works on API 23+)
```

---

## 🔍 VALIDATION RULES

### User Registration
```kotlin
object ValidationRules {
    
    const val MIN_NAME_LENGTH = 2
    const val MAX_NAME_LENGTH = 50
    const val MOBILE_LENGTH = 10
    const val MIN_PASSWORD_LENGTH = 8
    const val MAX_PASSWORD_LENGTH = 50
    
    // Name validation
    val NAME_REGEX = "^[a-zA-Z\\s]{2,50}$".toRegex()
    
    // Indian mobile number (starts with 6, 7, 8, or 9)
    val MOBILE_REGEX = "^[6-9][0-9]{9}$".toRegex()
    
    // Password (at least 1 letter and 1 number)
    val PASSWORD_REGEX = "^(?=.*[A-Za-z])(?=.*\\d).{8,50}$".toRegex()
    
    fun validateName(name: String): ValidationResult {
        return when {
            name.isBlank() -> ValidationResult.Error("Name cannot be empty")
            name.length < MIN_NAME_LENGTH -> ValidationResult.Error("Name must be at least $MIN_NAME_LENGTH characters")
            name.length > MAX_NAME_LENGTH -> ValidationResult.Error("Name must not exceed $MAX_NAME_LENGTH characters")
            !NAME_REGEX.matches(name) -> ValidationResult.Error("Name can only contain letters and spaces")
            else -> ValidationResult.Success
        }
    }
    
    fun validateMobile(mobile: String): ValidationResult {
        return when {
            mobile.isBlank() -> ValidationResult.Error("Mobile number cannot be empty")
            mobile.length != MOBILE_LENGTH -> ValidationResult.Error("Mobile number must be exactly $MOBILE_LENGTH digits")
            !MOBILE_REGEX.matches(mobile) -> ValidationResult.Error("Invalid mobile number format")
            else -> ValidationResult.Success
        }
    }
    
    fun validatePassword(password: String): ValidationResult {
        return when {
            password.isBlank() -> ValidationResult.Error("Password cannot be empty")
            password.length < MIN_PASSWORD_LENGTH -> ValidationResult.Error("Password must be at least $MIN_PASSWORD_LENGTH characters")
            password.length > MAX_PASSWORD_LENGTH -> ValidationResult.Error("Password must not exceed $MAX_PASSWORD_LENGTH characters")
            !PASSWORD_REGEX.matches(password) -> ValidationResult.Error("Password must contain at least one letter and one number")
            else -> ValidationResult.Success
        }
    }
}

sealed class ValidationResult {
    object Success : ValidationResult()
    data class Error(val message: String) : ValidationResult()
}
```

### Ticket ID Format
```kotlin
object TicketIdGenerator {
    
    private const val PREFIX = "NR"
    private const val DATE_FORMAT = "yyyyMMdd"
    private const val UNIQUE_ID_LENGTH = 5
    
    // Regex: NR-YYYYMMDD-XXXXX
    val TICKET_ID_REGEX = "^NR-\\d{8}-[A-Z0-9]{5}$".toRegex()
    
    fun generate(): String {
        val dateFormat = SimpleDateFormat(DATE_FORMAT, Locale.US).apply {
            timeZone = TimeZone.getTimeZone("UTC")
        }
        val datePart = dateFormat.format(Date())
        val uniquePart = UUID.randomUUID().toString()
            .replace("-", "")
            .take(UNIQUE_ID_LENGTH)
            .uppercase(Locale.US)
        
        return "$PREFIX-$datePart-$uniquePart"
    }
    
    fun validate(ticketId: String): Boolean {
        return TICKET_ID_REGEX.matches(ticketId)
    }
}
```

### GPS Coordinates
```kotlin
object GpsValidator {
    
    const val MIN_LATITUDE = -90.0
    const val MAX_LATITUDE = 90.0
    const val MIN_LONGITUDE = -180.0
    const val MAX_LONGITUDE = 180.0
    const val DECIMAL_PLACES = 4
    
    fun isValidLatitude(lat: Double): Boolean {
        return lat in MIN_LATITUDE..MAX_LATITUDE
    }
    
    fun isValidLongitude(lon: Double): Boolean {
        return lon in MIN_LONGITUDE..MAX_LONGITUDE
    }
    
    fun formatCoordinate(value: Double): String {
        return String.format(Locale.US, "%.${DECIMAL_PLACES}f", value)
    }
}
```

### Timestamp Format
```kotlin
object TimestampUtils {
    
    private const val ISO_8601_FORMAT = "yyyy-MM-dd'T'HH:mm:ss'Z'"
    
    fun getCurrentUtcTimestamp(): String {
        val dateFormat = SimpleDateFormat(ISO_8601_FORMAT, Locale.US).apply {
            timeZone = TimeZone.getTimeZone("UTC")
        }
        return dateFormat.format(Date())
    }
    
    fun parseTimestamp(timestamp: String): Date? {
        return try {
            val dateFormat = SimpleDateFormat(ISO_8601_FORMAT, Locale.US).apply {
                timeZone = TimeZone.getTimeZone("UTC")
            }
            dateFormat.parse(timestamp)
        } catch (e: Exception) {
            null
        }
    }
    
    fun formatForDisplay(timestamp: String): String {
        val date = parseTimestamp(timestamp) ?: return timestamp
        val displayFormat = SimpleDateFormat("dd MMM yyyy, hh:mm a", Locale.getDefault())
        return displayFormat.format(date)
    }
}
```

---

## 🚨 ERROR HANDLING (API 24+ COMPATIBLE)

### User-Facing Error Messages

```kotlin
object ErrorMessages {
    
    // Registration Errors
    const val ERROR_NAME_TOO_SHORT = "Name must be at least 2 characters"
    const val ERROR_MOBILE_INVALID_LENGTH = "Mobile number must be exactly 10 digits"
    const val ERROR_MOBILE_INVALID_FORMAT = "Mobile number must start with 6, 7, 8, or 9"
    const val ERROR_MOBILE_ALREADY_REGISTERED = "This mobile number is already registered"
    const val ERROR_PASSWORD_TOO_SHORT = "Password must be at least 8 characters"
    const val ERROR_PASSWORD_WEAK = "Password must contain at least one letter and one number"
    
    // Login Errors
    const val ERROR_MOBILE_NOT_REGISTERED = "Mobile number not registered"
    const val ERROR_PASSWORD_INCORRECT = "Incorrect password"
    const val ERROR_NETWORK = "Please check your internet connection"
    
    // Camera Errors
    const val ERROR_CAMERA_PERMISSION = "Camera permission is required to capture photos"
    const val ERROR_CAMERA_INIT_FAILED = "Failed to initialize camera. Please restart the app."
    const val ERROR_PHOTO_SAVE_FAILED = "Failed to save photo. Please try again."
    const val ERROR_CAMERA_NOT_AVAILABLE = "Camera is not available on this device"
    
    // GPS Errors
    const val ERROR_LOCATION_PERMISSION = "Location permission is required for GPS tagging"
    const val ERROR_GPS_UNAVAILABLE = "Unable to get current location. Please enable GPS."
    const val ERROR_GPS_DISABLED = "Location services are disabled. Please enable in Settings."
    
    // Gallery Errors
    const val ERROR_GALLERY_LOAD_FAILED = "Failed to load photo from gallery"
    const val ERROR_PHOTO_NO_GPS = "This photo does not contain location data. Using current device location."
    const val ERROR_PHOTO_TOO_LARGE = "Photo file is too large (max 10MB)"
    const val ERROR_PHOTO_UNSUPPORTED = "Unsupported image format"
    
    // Storage Errors (API specific)
    const val ERROR_STORAGE_PERMISSION = "Storage permission is required to access photos"
    const val ERROR_STORAGE_FULL = "Failed to save photo. Storage may be full."
    const val ERROR_STORAGE_ACCESS = "Unable to access photo storage"
    
    // Submission Errors
    const val ERROR_SUBMIT_FAILED = "Failed to submit report. Please try again."
    const val ERROR_MISSING_CLASSIFICATION = "Please select both issue type and severity"
    const val ERROR_NETWORK_SYNC = "Network error. Report saved locally and will sync later."
    
    // Tracker Errors
    const val ERROR_TICKET_ID_INVALID = "Invalid Ticket ID format. Expected: NR-YYYYMMDD-XXXXX"
    const val ERROR_TICKET_ID_NOT_FOUND = "Ticket ID not found. Please check and try again."
    
    // Permission Errors (API 24+ specific)
    fun getStoragePermissionError(): String {
        return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            "Photo access permission is required to upload from gallery"
        } else {
            "Storage permission is required to upload photos"
        }
    }
}
```

### Exception Handling Utility
```kotlin
object ExceptionHandler {
    
    fun handleCameraException(exception: Exception): String {
        return when (exception) {
            is CameraAccessException -> ErrorMessages.ERROR_CAMERA_NOT_AVAILABLE
            is IllegalStateException -> ErrorMessages.ERROR_CAMERA_INIT_FAILED
            is SecurityException -> ErrorMessages.ERROR_CAMERA_PERMISSION
            else -> ErrorMessages.ERROR_PHOTO_SAVE_FAILED
        }
    }
    
    fun handleStorageException(exception: Exception): String {
        return when (exception) {
            is IOException -> ErrorMessages.ERROR_STORAGE_FULL
            is SecurityException -> ErrorMessages.ERROR_STORAGE_PERMISSION
            is FileNotFoundException -> ErrorMessages.ERROR_STORAGE_ACCESS
            else -> ErrorMessages.ERROR_PHOTO_SAVE_FAILED
        }
    }
    
    fun handleNetworkException(exception: Exception): String {
        return when (exception) {
            is UnknownHostException -> ErrorMessages.ERROR_NETWORK
            is SocketTimeoutException -> "Connection timeout. Please try again."
            is SSLException -> "Secure connection failed. Please check your network."
            else -> ErrorMessages.ERROR_NETWORK
        }
    }
}
```

---

## 📊 ANALYTICS & LOGGING

### Key Events to Track

```kotlin
object AnalyticsEvents {
    
    // User Events
    const val EVENT_USER_REGISTERED = "user_registered"
    const val EVENT_USER_LOGGED_IN = "user_logged_in"
    const val EVENT_USER_LOGGED_OUT = "user_logged_out"
    
    // Report Events
    const val EVENT_REPORT_CAPTURED_CAMERA = "report_captured_camera"
    const val EVENT_REPORT_CAPTURED_GALLERY = "report_captured_gallery"
    const val EVENT_REPORT_SUBMITTED = "report_submitted"
    const val EVENT_REPORT_SUBMISSION_FAILED = "report_submission_failed"
    
    // UI Events
    const val EVENT_SCREEN_VIEWED = "screen_viewed"
    const val EVENT_BUTTON_CLICKED = "button_clicked"
    const val EVENT_PERMISSION_REQUESTED = "permission_requested"
    const val EVENT_PERMISSION_GRANTED = "permission_granted"
    const val EVENT_PERMISSION_DENIED = "permission_denied"
    
    // Error Events
    const val EVENT_ERROR_OCCURRED = "error_occurred"
    const val EVENT_CAMERA_ERROR = "camera_error"
    const val EVENT_GPS_ERROR = "gps_error"
    
    // Performance Events
    const val EVENT_EXIF_EXTRACTION_TIME = "exif_extraction_time"
    const val EVENT_PHOTO_SAVE_TIME = "photo_save_time"
    const val EVENT_DB_QUERY_TIME = "db_query_time"
    
    // Parameters
    const val PARAM_USER_ID = "user_id"
    const val PARAM_TICKET_ID = "ticket_id"
    const val PARAM_ISSUE_TYPE = "issue_type"
    const val PARAM_SEVERITY = "severity"
    const val PARAM_IS_FROM_GALLERY = "is_from_gallery"
    const val PARAM_HAS_EXIF_GPS = "has_exif_gps"
    const val PARAM_SCREEN_NAME = "screen_name"
    const val PARAM_BUTTON_NAME = "button_name"
    const val PARAM_PERMISSION_TYPE = "permission_type"
    const val PARAM_ERROR_MESSAGE = "error_message"
    const val PARAM_ANDROID_VERSION = "android_version"
    const val PARAM_DURATION_MS = "duration_ms"
}
```

### Logging Standards (API 24+ Compatible)

```kotlin
import android.util.Log
import com.nammaraste.reporter.BuildConfig

object Logger {
    
    private const val TAG_PREFIX = "NammaRaste"
    
    fun v(tag: String, message: String) {
        if (BuildConfig.DEBUG) {
            Log.v("$TAG_PREFIX:$tag", message)
        }
    }
    
    fun d(tag: String, message: String) {
        if (BuildConfig.DEBUG) {
            Log.d("$TAG_PREFIX:$tag", message)
        }
    }
    
    fun i(tag: String, message: String) {
        Log.i("$TAG_PREFIX:$tag", message)
    }
    
    fun w(tag: String, message: String, throwable: Throwable? = null) {
        if (throwable != null) {
            Log.w("$TAG_PREFIX:$tag", message, throwable)
        } else {
            Log.w("$TAG_PREFIX:$tag", message)
        }
    }
    
    fun e(tag: String, message: String, throwable: Throwable? = null) {
        if (throwable != null) {
            Log.e("$TAG_PREFIX:$tag", message, throwable)
        } else {
            Log.e("$TAG_PREFIX:$tag", message)
        }
    }
    
    // Performance logging
    fun logPerformance(tag: String, operation: String, durationMs: Long) {
        if (BuildConfig.DEBUG) {
            Log.d("$TAG_PREFIX:Perf:$tag", "$operation took ${durationMs}ms")
        }
    }
    
    // NEVER log sensitive data
    fun logUserAction(tag: String, action: String, userId: String) {
        // Only log hashed/anonymized user ID
        val anonymizedId = userId.take(8) + "****"
        Log.i("$TAG_PREFIX:$tag", "User $anonymizedId performed: $action")
    }
}

// Usage in classes
class CameraFragment : Fragment() {
    companion object {
        private const val TAG = "CameraFragment"
    }
    
    private fun initializeCamera() {
        Logger.d(TAG, "Initializing camera with API ${Build.VERSION.SDK_INT}")
    }
    
    private fun handleError(exception: Exception) {
        Logger.e(TAG, "Camera error occurred", exception)
    }
}
```

---

## ✅ SUCCESS CRITERIA (API 24+ TESTED)

### SC-01: 3-Click Rule Compliance
**Live Camera Capture:**
1. Click shutter → Photo captured
2. Select issue type + severity → Classification complete
3. Tap submit → Report submitted

**Gallery Upload:**
1. Click gallery → Photo picker/chooser opened
2. Select photo → Photo selected
3. Tap submit → Report submitted

**Verification:** Manual QA on API 24, 29, and 33+ devices

### SC-02: Unique Ticket IDs
**Requirement:** Every submitted report must have a unique Ticket ID

**Verification:**
```kotlin
@Test
fun testUniqueTicketIds() = runTest {
    val ids = List(50) { TicketIdGenerator.generate() }
    assertEquals(50, ids.distinct().size)
    assertTrue(ids.all { TicketIdGenerator.validate(it) })
}
```

### SC-03: GPS Accuracy
**Requirement:** GPS coordinates within 50 meters of actual location

**Verification:** Field test with known coordinates on API 24+ devices

### SC-04: EXIF Extraction Performance
**Requirement:** Metadata extraction completes in <500ms

**Verification:**
```kotlin
@Test
fun testExifExtractionPerformance() = runTest {
    val startTime = System.currentTimeMillis()
    val metadata = metadataHelper.extractExifMetadata(testPhotoUri)
    val duration = System.currentTimeMillis() - startTime
    assertTrue("EXIF extraction took ${duration}ms", duration < 500)
}
```

### SC-05: API 24 Compatibility
**Requirements:**
- App installs and runs on Android 7.0 (API 24)
- All features work correctly on API 24-36
- Storage permissions handled correctly per API level
- No crashes on low-end devices (2GB RAM, Snapdragon 425)

**Verification:**
- Test on physical API 24 device or emulator
- Test storage permissions on API 24, 29, and 33+
- Verify EXIF extraction works across all API levels
- Check memory usage stays under 150MB

### SC-06: Build Performance
**Requirement:** Clean build completes in <3 minutes (with KSP)

**Verification:**
```bash
# Clean build test
./gradlew clean assembleDebug --profile

# Check build/reports/profile/ for build scan
```

### SC-07: Security Compliance
**Requirements:**
- Passwords stored as bcrypt hash only (never plaintext)
- Session tokens in EncryptedSharedPreferences (AES256_GCM)
- No sensitive data in logs
- ProGuard/R8 enabled for release builds
- Network security config enforces HTTPS

**Verification:** 
- Code review
- Security audit
- APK analysis with apkanalyzer
- Check for plaintext in decompiled APK

### SC-08: Simplified Authentication
**Requirements:**
- No profile photo upload in UI or database
- No email field in registration
- No DigiLocker integration
- Only name + mobile + password

**Verification:** Code review + UI inspection on all screens

### SC-09: No Social Sharing
**Requirements:**
- No "Share via WhatsApp" button
- No generic share intent
- Only "Copy Ticket ID" available

**Verification:** UI inspection + navigation flow test

---

## 🛡️ SECURITY BEST PRACTICES (API 24+ COMPATIBLE)

### Password Storage
```kotlin
import org.mindrot.jbcrypt.BCrypt

object PasswordUtils {
    
    private const val BCRYPT_ROUNDS = 10
    
    fun hashPassword(plainPassword: String): String {
        return BCrypt.hashpw(plainPassword, BCrypt.gensalt(BCRYPT_ROUNDS))
    }
    
    fun verifyPassword(plainPassword: String, hashedPassword: String): Boolean {
        return try {
            BCrypt.checkpw(plainPassword, hashedPassword)
        } catch (e: Exception) {
            Logger.e("PasswordUtils", "Password verification failed", e)
            false
        }
    }
}
```

### Session Management (API 24+ Compatible)
```kotlin
import androidx.security.crypto.EncryptedSharedPreferences
import androidx.security.crypto.MasterKey

object SessionManager {
    
    private const val PREFS_NAME = "namma_raste_secure_prefs"
    private const val KEY_USER_ID = "current_user_id"
    
    fun getEncryptedPrefs(context: Context): SharedPreferences {
        val masterKey = MasterKey.Builder(context)
            .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
            .build()
        
        return EncryptedSharedPreferences.create(
            context,
            PREFS_NAME,
            masterKey,
            EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
            EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
        )
    }
    
    fun saveUserId(context: Context, userId: String) {
        getEncryptedPrefs(context).edit()
            .putString(KEY_USER_ID, userId)
            .apply()
    }
    
    fun getUserId(context: Context): String? {
        return getEncryptedPrefs(context).getString(KEY_USER_ID, null)
    }
    
    fun clearSession(context: Context) {
        getEncryptedPrefs(context).edit()
            .clear()
            .apply()
    }
    
    fun isLoggedIn(context: Context): Boolean {
        return getUserId(context) != null
    }
}
```

### ProGuard Rules (proguard-rules.pro) - API 24+ Compatible
```proguard
# General optimizations
-optimizationpasses 5
-dontusemixedcaseclassnames
-dontskipnonpubliclibraryclasses
-verbose
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*

# Keep Room entities
-keep class com.nammaraste.reporter.data.local.entities.** { *; }
-keepclassmembers class com.nammaraste.reporter.data.local.entities.** { *; }

# Keep BCrypt
-keep class org.mindrot.jbcrypt.** { *; }
-dontwarn org.mindrot.jbcrypt.**

# Keep Firebase
-keep class com.google.firebase.** { *; }
-dontwarn com.google.firebase.**
-keepattributes *Annotation*

# Keep Hilt generated classes
-keep class dagger.hilt.** { *; }
-keep class javax.inject.** { *; }
-keep class * extends dagger.hilt.android.internal.managers.ViewComponentManager$FragmentContextWrapper { *; }
-keepclassmembers class * {
    @dagger.hilt.** *;
}

# Keep CameraX
-keep class androidx.camera.** { *; }
-dontwarn androidx.camera.**

# Keep Coil image loading
-keep class coil3.** { *; }
-dontwarn coil3.**

# Keep EXIF Interface
-keep class androidx.exifinterface.media.ExifInterface { *; }
-keepclassmembers class androidx.exifinterface.media.ExifInterface { *; }

# Keep Kotlin metadata
-keep class kotlin.Metadata { *; }
-keepattributes RuntimeVisibleAnnotations
-keepattributes RuntimeVisibleParameterAnnotations
-keepattributes AnnotationDefault

# Keep Navigation Component
-keepnames class androidx.navigation.fragment.NavHostFragment
-keep class * extends androidx.fragment.app.Fragment{}

# Keep data classes (for JSON serialization if using Gson/Moshi)
-keep class com.nammaraste.reporter.data.models.** { *; }

# Remove all Log calls in release (except ERROR)
-assumenosideeffects class android.util.Log {
    public static *** v(...);
    public static *** d(...);
    public static *** i(...);
    public static *** w(...);
}

# Keep line numbers for debugging stack traces
-keepattributes SourceFile,LineNumberTable
-renamesourcefileattribute SourceFile

# Crashlytics (if enabled)
-keepattributes *Annotation*
-keep class com.crashlytics.** { *; }
-dontwarn com.crashlytics.**
```

---

## 📚 GLOSSARY (UPDATED FOR API 24+)

### Technical Terms

**AGP (Android Gradle Plugin):** Build system plugin for Android projects (version 9.1.0)

**API Level 24:** Android 7.0 Nougat - Minimum supported version for this app

**KSP (Kotlin Symbol Processing):** Modern replacement for kapt, 2-3x faster annotation processing

**Version Catalog (libs.versions.toml):** Modern dependency management approach in Gradle

**Desugaring:** Process that allows using Java 21 features on API 24+ devices

**CameraX:** Jetpack library for consistent camera API across devices (version 1.4.1)

**Room:** Android persistence library (ORM) for SQLite (version 2.6.1)

**Hilt:** Dependency injection framework built on Dagger (version 2.54)

**MVVM:** Model-View-ViewModel architectural pattern

**StateFlow:** Kotlin Coroutines-based reactive state holder (preferred over LiveData)

**LiveData:** Legacy lifecycle-aware observable (still supported, but StateFlow preferred)

**ExifInterface:** AndroidX library for reading image metadata

**EXIF:** Exchangeable Image File Format - metadata in photos

**GPS Redaction:** Android 10+ privacy feature stripping GPS from photos

**EncryptedSharedPreferences:** Jetpack Security library for encrypted storage (works on API 23+)

**bcrypt:** Cryptographic hashing for passwords

**SafeArgs:** Navigation Component plugin for type-safe arguments

**ktlint:** Kotlin linting tool

**Scoped Storage:** Android 10+ storage model

**Photo Picker:** Android 13+ system UI for photo selection

**FileProvider:** Android component for secure file sharing

### Android Version Codenames

- **API 24-25:** Nougat (7.0-7.1)
- **API 26-27:** Oreo (8.0-8.1)
- **API 28:** Pie (9.0)
- **API 29:** Q / Android 10
- **API 30:** R / Android 11
- **API 31-32:** S / Android 12-12L
- **API 33:** Tiramisu / Android 13
- **API 34:** Upside Down Cake / Android 14
- **API 35:** Vanilla Ice Cream / Android 15
- **API 36:** Android 16 (current target)

---

## 📝 CHANGELOG

### Version 2.3 (Current - API 24 Support)

**Updated:**
- ✅ Minimum SDK: API 26 → API 24 (Android 7.0 Nougat)
- ✅ Android Studio: Ladybug → Panda | 2025.3.1
- ✅ Android Gradle Plugin: 9.0.1 → 9.1.0
- ✅ Kotlin: 2.0.21 → 2.1.0
- ✅ Gradle: 8.9 → 9.0
- ✅ Java: JDK 17 → JDK 21
- ✅ Target SDK: 35 → 36 (Android 16)
- ✅ Compile SDK: 35 → 36
- ✅ AndroidX Core: 1.17.0 → 1.15.0 (API 24 compatible)
- ✅ Lifecycle: 2.10.0 → 2.8.7
- ✅ Activity: 1.12.4 → 1.9.3
- ✅ Fragment: Added 1.8.5
- ✅ Coil: 2.7.0 → 3.0.4 (Coil 3 with OkHttp)
- ✅ Coroutines: 1.9.0 → 1.10.1
- ✅ KSP: 2.0.21-1.0.28 → 2.1.0-1.0.29
- ✅ Firebase BOM: 33.7.0 → 33.8.0
- ✅ Hilt: 2.52 → 2.54

**Added:**
- ✅ Desugaring support for Java 21 features on API 24+
- ✅ Version catalog (libs.versions.toml) for dependency management
- ✅ Network security config XML
- ✅ API 24-specific permission handling
- ✅ CompatibilityUtils for multi-API support
- ✅ StorageUtils with API 24-36 compatibility
- ✅ Enhanced ProGuard rules for API 24+
- ✅ Multi-API level testing strategy

**API 24 Specific Changes:**
- ✅ WRITE_EXTERNAL_STORAGE permission for API 24-28
- ✅ Legacy photo picker for API 24-32
- ✅ Compatibility layer for storage access
- ✅ Backward-compatible EXIF extraction
- ✅ API-level specific error messages

**Removed:**
- ❌ Dependencies not compatible with API 24

---

## 🔄 VERSION CONTROL

**Last Updated:** 2025-01-20  
**Document Version:** 2.3  
**Android Studio Version:** Panda | 2025.3.1  
**AGP Version:** 9.1.0  
**Kotlin Version:** 2.1.0  
**Minimum API Level:** 24 (Android 7.0)  
**Target API Level:** 36 (Android 16)  
**Maintainer:** Development Team

---

## 📝 FINAL NOTES

This SOP represents the **complete technical specification** for the Namma-Raste Reporter Android application optimized for:

### Core Principles
1. **Wide Compatibility:** Supports Android 7.0 (API 24) through Android 16 (API 36)
2. **Modern Tooling:** Android Studio Panda 2025.3.1, Kotlin 2.1.0, AGP 9.1.0
3. **Performance First:** KSP for faster builds, version catalog for dependency management
4. **Simplicity:** Minimal user data (name, mobile, password only)
5. **Privacy Focused:** No external sharing, API-appropriate storage handling

### Technical Highlights
- **100% Kotlin 2.1.0** with K2 compiler and MVVM + Clean Architecture
- **API 24-36 Support** with proper compatibility layers
- **KSP instead of kapt** for 2-3x faster annotation processing
- **Version Catalog** for centralized dependency management
- **Java 21 with Desugaring** for modern language features on older devices
- **Dual Input:** Live camera + gallery with version-specific implementations
- **Local-First:** Room database with optional Firebase sync
- **Security:** BCrypt hashing, EncryptedSharedPreferences (API 23+ compatible)
- **Performance:** <500ms EXIF extraction, ≥30fps camera preview
- **Compatibility:** Tested on Android 7.0 through Android 16

### Out of Scope (Deliberately Excluded)
- Profile photo management
- Email-based authentication
- DigiLocker/Aadhaar integration
- Social media sharing
- Compose UI (using XML Views for API 24 compatibility)
- Payment/rewards system


---