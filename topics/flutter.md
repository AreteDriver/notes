# Flutter Patterns

Flutter and Dart development patterns.

---

## Firebase Integration

### Service Pattern with Auth State

Services should listen to auth state changes and refresh data when user changes:

```dart
class MyService extends ChangeNotifier {
  final FirebaseAuth _auth = FirebaseAuth.instance;
  final FirebaseFirestore _firestore = FirebaseFirestore.instance;

  StreamSubscription<User?>? _authSubscription;
  StreamSubscription<QuerySnapshot>? _dataSubscription;

  MyService() {
    _authSubscription = _auth.authStateChanges().listen((user) {
      if (user != null) {
        _loadFromFirestore();
      } else {
        _dataSubscription?.cancel();
        _clearData();
      }
    });
  }

  String? get _userId => _auth.currentUser?.uid;

  @override
  void dispose() {
    _authSubscription?.cancel();
    _dataSubscription?.cancel();
    super.dispose();
  }
}
```

### Real-time Firestore Listeners

```dart
_dataSubscription = _firestore
    .collection('items')
    .where('userId', isEqualTo: userId)
    .orderBy('createdAt', descending: true)
    .snapshots()
    .listen((snapshot) {
  _items = snapshot.docs.map((doc) {
    final data = doc.data();
    data['id'] = doc.id;  // Include document ID
    return Item.fromJson(data);
  }).toList();
  notifyListeners();
}, onError: (e) {
  debugPrint('Firestore error: $e');
  _loadFallbackData();
});
```

### Offline-First with Local Cache

```dart
// Save to both Firestore and local cache
await Future.wait([
  _saveToLocal(),      // SharedPreferences for offline
  _saveToFirestore(),  // Cloud sync
]);
```

---

## Provider + ChangeNotifier

### Multi-Provider Setup

```dart
return MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (_) => AuthService()),
    ChangeNotifierProvider(create: (_) => UserPreferencesService()),
    ChangeNotifierProvider(create: (_) => DataService()),
  ],
  child: MaterialApp(...),
);
```

### Consumer Pattern

```dart
// Single service
Consumer<AuthService>(
  builder: (context, auth, _) {
    if (!auth.isAuthenticated) return SignInScreen();
    return MainScreen();
  },
)

// Multiple services
Consumer2<AuthService, UserPreferencesService>(
  builder: (context, auth, prefs, _) {
    // Access both services
  },
)
```

---

## Auth Gate Pattern

```dart
class AppNavigator extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer2<AuthService, UserPreferencesService>(
      builder: (context, auth, prefs, _) {
        if (auth.isLoading || prefs.isLoading) {
          return LoadingScreen();
        }
        if (!auth.isAuthenticated) {
          return SignInScreen();
        }
        if (!prefs.hasCompletedOnboarding) {
          return OnboardingFlow();
        }
        return MainScreen();
      },
    );
  }
}
```

---

## Cloud Functions

```dart
class CloudFunctionsService {
  final _functions = FirebaseFunctions.instanceFor(region: 'us-central1');

  Future<Recipe> generateRecipe(Map<String, dynamic> params) async {
    final callable = _functions.httpsCallable(
      'generateRecipe',
      options: HttpsCallableOptions(timeout: Duration(seconds: 60)),
    );
    final result = await callable.call(params);
    return Recipe.fromJson(result.data);
  }
}
```

---

## Project Setup

### Generate Platform Files

If only `lib/` exists (Dart-only project):

```bash
cd mobile
flutter create .  # Generates android/, ios/, etc.
```

### Firebase Config Files

1. Download from Firebase Console
2. Place in correct locations:
   - `android/app/google-services.json`
   - `ios/Runner/GoogleService-Info.plist`

### Common Dependencies

```yaml
# pubspec.yaml
dependencies:
  # State
  provider: ^6.1.1

  # Firebase
  firebase_core: ^3.8.0
  firebase_auth: ^5.3.0
  cloud_firestore: ^5.5.0
  cloud_functions: ^5.1.0
  google_sign_in: ^6.2.0

  # Storage
  shared_preferences: ^2.2.2
```

---

## Common Issues

### gitignore Blocking lib/

Python's `.gitignore` uses `lib/` which blocks Flutter's `mobile/lib/`.

**Fix:** Change `lib/` to `/lib/` (root-only match).

### Firebase Init Error

Ensure `WidgetsFlutterBinding.ensureInitialized()` before `Firebase.initializeApp()`:

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MyApp());
}
```

---

## iOS Build Preparation

### Setup Script Pattern

Create `scripts/setup-ios.sh` for Mac automation:
- Check prerequisites (flutter, xcode, cocoapods)
- Generate platform files (`flutter create .`)
- Configure bundle ID
- Prompt for Firebase config
- Auto-configure Google Sign-In URL scheme from plist
- Install dependencies

### Makefile Targets

```makefile
setup:        # Run setup script
deps:         # flutter pub get
deps-ios:     # pod install
build-ios:    # flutter build ios --debug
release-ios:  # flutter build ios --release
run-ios:      # flutter run -d ios
xcode:        # open ios/Runner.xcworkspace
clean:        # flutter clean
```

### Key iOS Configuration

```bash
# Minimum iOS for Firebase
# In ios/Podfile:
platform :ios, '13.0'

# Google Sign-In URL scheme (in Info.plist)
# Use REVERSED_CLIENT_ID from GoogleService-Info.plist
```

### Xcode Signing Quick Reference

| Scenario | Action |
|----------|--------|
| Simulator only | Uncheck "Automatically manage signing" |
| Device testing | Check auto-sign, select free team |
| App Store | Check auto-sign, select paid team ($99/yr) |

---

*Last updated: 2026-01-19*
