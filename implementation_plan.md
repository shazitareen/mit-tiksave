# Implementation Plan: Remote Config & Dynamic Splash Loading Screen

This plan details the steps to implement a centralized Remote Config/Ad display control system using a utility class (`AdsUtils`), a premium 5-6 second splash loading experience that visually displays the status of the fetched configurations, and the update of all ad-related conditions to strictly follow the retrieved values.

---

## User Review Required

> [!IMPORTANT]
> **Key Design Decisions:**
> 1. **Centralized Configuration (`AdsUtils`):** We will create a `lib/utils/ads_utils.dart` class to keep track of the active/inactive states of the ads.
> 2. **Splash Screen Loading Experience:** We will update the `SplashScreen` to show a beautiful, modern progress loader for **5 to 6 seconds** while fetching Remote Config.
> 3. **Debug Config Viewer on Splash Screen (Optional but recommended for testing):** To meet the client's request of *"Show remote config data fetched from firebase on a splash screen"*, we will render a sleek, premium grid indicating the status of each ad (e.g. `Rewarded Ads: Active ✅`, etc.).
> 4. **Testing Parameters:** We will add instructions on how the user can test different boolean configurations (e.g. setting them to `false` to verify ads are bypassed immediately).

---

## Proposed Changes

### 1. [NEW] Utility Class `AdsUtils`
We will create a new file `lib/utils/ads_utils.dart` to store global static configurations initialized to `true` by default.

#### [NEW] [ads_utils.dart](file:///d:/android%20projects/Client%20TT%20downloader%20-%20TikTokio/lib/utils/ads_utils.dart)
```dart
import '../services/firebase_service.dart';

class AdsUtils {
  static bool showrewarded = true;
  static bool homebanner = true;
  static bool showappopen = true;
  static bool shownative = true;
  static bool Showsplashinter = true;
  static bool Showinter = true;
  static int interstitialClickInterval = 3;

  /// Syncs global settings from successful Firebase Remote Config fetch
  static void updateFromRemoteConfig(FirebaseService firebaseService) {
    showrewarded = firebaseService.showrewarded;
    homebanner = firebaseService.homebanner;
    showappopen = firebaseService.showappopen;
    shownative = firebaseService.shownative;
    Showsplashinter = firebaseService.Showsplashinter;
    Showinter = firebaseService.Showinter;
    interstitialClickInterval = firebaseService.interstitialClickInterval;
  }
}
```

---

### 2. [MODIFY] [firebase_service.dart](file:///d:/android%20projects/Client%20TT%20downloader%20-%20TikTokio/lib/services/firebase_service.dart)
We will modify the Firebase service to:
- Set default remote config booleans to `true` (as requested).
- Separate the initialization (`init()`) from the actual fetch operation (`fetchRemoteConfig()`) so the splash screen can trigger it and display the results dynamically.

```diff
       await remoteConfig.setDefaults(const {
-        'showrewarded': false, // Rewarded video ads (for downloading)
-        'homebanner': false, // Collapsible banner ad
-        'showappopen': false, // App open ad on launch
-        'shownative': false, // Native ads in history list
-        'Showsplashinter': false, // Splash interstitial ad
-        'Showinter': false, // Click-count interstitial ad
+        'showrewarded': true, // Rewarded video ads (for downloading)
+        'homebanner': true, // Collapsible banner ad
+        'showappopen': true, // App open ad on launch
+        'shownative': true, // Native ads in history list
+        'Showsplashinter': true, // Splash interstitial ad
+        'Showinter': true, // Click-count interstitial ad
         'interstitial_click_interval': 3,
       });
```

---

### 3. [MODIFY] [ad_manager.dart](file:///d:/android%20projects/Client%20TT%20downloader%20-%20TikTokio/lib/services/ad_manager.dart)
We will replace all references to `FirebaseService()` inside `ad_manager.dart` with `AdsUtils` to use the values parsed during the splash screen loader.

---

### 4. [MODIFY] [splash_screen.dart](file:///d:/android%20projects/Client%20TT%20downloader%20-%20TikTokio/lib/screens/splash_screen.dart)
We will rebuild the splash screen to:
- Show a premium 5.5-second loading indicator (animating from "Initializing..." to "Configurations Loaded!").
- Call `FirebaseService().fetchRemoteConfig()` using `async/await`.
- Render a sleek UI panel showing the fetched configuration status dynamically.
- Check `AdsUtils` instead of `FirebaseService()` to decide whether to launch the app open ad or splash interstitial.

---

### 5. [MODIFY] Screens checking Ads
We will update the following screens to query `AdsUtils` instead of `FirebaseService()` directly:
- [result_screen.dart](file:///d:/android%20projects/Client%20TT%20downloader%20-%20TikTokio/lib/screens/result_screen.dart)
- [main_navigation_screen.dart](file:///d:/android%20projects/Client%20TT%20downloader%20-%20TikTokio/lib/screens/main_navigation_screen.dart)
- [history_screen.dart](file:///d:/android%20projects/Client%20TT%20downloader%20-%20TikTokio/lib/screens/history_screen.dart)
- [help_screen.dart](file:///d:/android%20projects/Client%20TT%20downloader%20-%20TikTokio/lib/screens/help_screen.dart)

---

## Verification Plan

### Manual Verification & Testing Guidelines

> [!TIP]
> **How to Test Remote Config Toggles:**
> For testing purposes, you can configure these values in your Firebase Remote Config Console or modify the default values inside `lib/services/firebase_service.dart`.
> 
> **To test bypassing ads (All ads off):**
> Set the parameters in Firebase or defaults in `firebase_service.dart` to `false`:
> ```dart
> 'showrewarded': false,
> 'homebanner': false,
> 'showappopen': false,
> 'shownative': false,
> 'Showsplashinter': false,
> 'Showinter': false,
> ```
> Expected behavior:
> - **Rewarded Ad Bypass:** Tapping "Download Video" will instantly trigger the download without showing the "Watch Ad" popup dialog.
> - **Banner Ad Bypass:** The banner space at the bottom of the Home / Navigation / Help screen will be empty/gone.
> - **App Open Bypass:** Opening the app from cold start will skip the App Open ad entirely and launch home instantly.
> - **Native Ad Bypass:** History screen will display download cards with no native ads.

1. **Verify 5-6 Second Splash Screen Delay:** Ensure the splash screen doesn't navigate instantly and gives enough time to fetch Remote Config.
2. **Verify Configuration Render on Splash Screen:** Check that a professional status list/widget displays the active ad modules.
