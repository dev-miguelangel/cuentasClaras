# Plan: CuentasRápidas — Flutter (Android+iOS) + Angular (Web) + Firebase

## Context
Rebuild the current single-HTML expense tracker into a production-quality cross-platform app with:
- Flutter for Android & iOS (native performance, offline-first)
- Angular for web (existing user base, desktop + mobile browser)
- Shared Firebase backend: Firestore + Google Authentication
- Feature parity with current HTML app (Kanban boards, budgets, priorities, analytics)

---

## Architecture

### Monorepo structure
```
cuentasrapidas/
├── flutter_app/          # Flutter (Android + iOS)
├── angular_app/          # Angular (Web)
├── firebase/             # Firestore rules, indexes, emulator config
└── shared/               # Shared constants / data schemas (optional)
```

### Firebase Project
- **Auth**: Google Sign-In provider enabled
- **Firestore**: Native mode, region `us-central1`
- **Hosting**: Angular app deployed via `firebase deploy --only hosting`

---

## Firestore Data Schema

```
/users/{uid}/
  boards/{boardId}/
    name: string
    createdAt: timestamp
    expenses/{expenseId}/
      titulo: string
      monto: number
      estado: "porPagar" | "carrito" | "pagado"
      prioridad: "vital" | "gusto" | "capricho"
      createdAt: timestamp
  budgets/{boardId}/
    amount: number
```

---

## Firestore Security Rules

```firestore
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isOwner(uid) {
      return request.auth != null && request.auth.uid == uid;
    }
    match /users/{uid}/{document=**} {
      allow read, write: if isOwner(uid);
    }
  }
}
```

---

## Flutter App

### Dependencies (`pubspec.yaml`)
```yaml
dependencies:
  flutter_riverpod: ^2.5
  riverpod_annotation: ^2.3
  go_router: ^14.0
  firebase_core: ^3.0
  firebase_auth: ^5.0
  cloud_firestore: ^5.0
  google_sign_in: ^6.2
  fl_chart: ^0.68
  flutter_slidable: ^3.1
  intl: ^0.19
  freezed_annotation: ^2.4

dev_dependencies:
  build_runner: ^2.4
  freezed: ^2.4
  riverpod_generator: ^2.3
```

### Feature folder structure
```
lib/
├── main.dart
├── firebase_options.dart          # FlutterFire CLI generated
├── core/
│   ├── router.dart                # GoRouter config
│   └── theme.dart
├── features/
│   ├── auth/
│   │   ├── auth_provider.dart     # Riverpod StreamProvider for authStateChanges
│   │   └── login_screen.dart      # Google Sign-In button
│   ├── boards/
│   │   ├── boards_provider.dart   # Stream<List<Board>> from Firestore
│   │   ├── boards_screen.dart     # Board list + create/delete
│   │   └── board_model.dart       # Freezed model
│   ├── expenses/
│   │   ├── expenses_provider.dart # Stream<List<Expense>> per board
│   │   ├── kanban_screen.dart     # 3 columns + swipe gestures
│   │   ├── expense_card.dart      # Slidable card with move/delete
│   │   ├── add_expense_sheet.dart # Bottom sheet form (stays open)
│   │   └── expense_model.dart     # Freezed model
│   ├── budget/
│   │   └── budget_provider.dart   # budget per board
│   └── analytics/
│       └── analytics_screen.dart  # fl_chart donuts per column
```

### Google Sign-In (Flutter)
```dart
// features/auth/auth_provider.dart
final authProvider = StreamProvider<User?>((ref) {
  return FirebaseAuth.instance.authStateChanges();
});

Future<void> signInWithGoogle() async {
  final googleUser = await GoogleSignIn().signIn();
  final googleAuth = await googleUser!.authentication;
  final credential = GoogleAuthProvider.credential(
    accessToken: googleAuth.accessToken,
    idToken: googleAuth.idToken,
  );
  await FirebaseAuth.instance.signInWithCredential(credential);
}
```

### GoRouter config
```dart
final router = GoRouter(
  redirect: (context, state) {
    final isLoggedIn = ref.read(authProvider).value != null;
    if (!isLoggedIn) return '/login';
    return null;
  },
  routes: [
    GoRoute(path: '/login', builder: (_, __) => LoginScreen()),
    GoRoute(path: '/boards', builder: (_, __) => BoardsScreen()),
    GoRoute(path: '/boards/:boardId', builder: (_, state) =>
      KanbanScreen(boardId: state.pathParameters['boardId']!)),
    GoRoute(path: '/boards/:boardId/analytics', builder: (_, state) =>
      AnalyticsScreen(boardId: state.pathParameters['boardId']!)),
  ],
);
```

---

## Angular App

### Dependencies
```bash
ng new angular_app --standalone --routing --style=scss
ng add @angular/material
npm install @angular/fire firebase
npm install ng2-charts chart.js
```

### Key files
```
src/app/
├── app.config.ts              # provideFirebaseApp, provideAuth, provideFirestore
├── app.routes.ts              # lazy-loaded routes
├── core/
│   ├── auth.service.ts        # Google Sign-In via signInWithPopup
│   └── auth.guard.ts          # functional guard
├── features/
│   ├── auth/
│   │   └── login/login.component.ts
│   ├── boards/
│   │   └── boards/boards.component.ts
│   └── kanban/
│       ├── kanban/kanban.component.ts    # CDK DragDrop between columns
│       ├── expense-card/
│       ├── add-expense/
│       ├── budget/
│       └── analytics/
```

### Google Sign-In (Angular)
```typescript
// core/auth.service.ts
@Injectable({ providedIn: 'root' })
export class AuthService {
  private auth = inject(Auth);
  user$ = authState(this.auth);

  async signInWithGoogle() {
    const provider = new GoogleAuthProvider();
    await signInWithPopup(this.auth, provider);
  }

  signOut() { return signOut(this.auth); }
}
```

### app.config.ts
```typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideFirebaseApp(() => initializeApp(environment.firebase)),
    provideAuth(() => getAuth()),
    provideFirestore(() => getFirestore()),
  ]
};
```

### Kanban with CDK DragDrop
```typescript
// kanban.component.ts
drop(event: CdkDragDrop<Expense[]>) {
  if (event.previousContainer !== event.container) {
    const expense = event.previousContainer.data[event.previousIndex];
    const newEstado = event.container.id as Estado;
    this.expenseService.updateEstado(expense.id, newEstado);
  }
}
```

---

## Implementation Order (10 days)

| Day | Task |
|-----|------|
| 1 | Firebase project setup: enable Auth + Firestore, deploy security rules, configure emulators |
| 2 | Flutter: FlutterFire CLI init, auth flow (Google Sign-In → boards list) |
| 3 | Flutter: Firestore CRUD for boards + expenses, Riverpod providers |
| 4 | Flutter: Kanban UI (3 columns, swipe to move, add-expense bottom sheet) |
| 5 | Flutter: Budget screen + Analytics screen (fl_chart donuts) |
| 6 | Angular: project scaffold, Firebase config, auth service + guard |
| 7 | Angular: boards list + kanban with CDK DragDrop |
| 8 | Angular: budget component + analytics charts (ng2-charts) |
| 9 | Angular: deploy to Firebase Hosting, test auth redirect |
| 10 | Polish: offline support (Firestore enablePersistence), error states, loading skeletons |

---

## Verification
1. Run Firebase emulators: `firebase emulators:start`
2. Flutter: `flutter run` on iOS + Android simulators → sign in with Google → create board → add expense → verify Firestore document created
3. Angular: `ng serve` → sign in → drag card between columns → verify Firestore updated
4. Security: attempt unauthenticated Firestore write → must be denied
5. Deploy: `firebase deploy` → verify hosted Angular app loads and auth works in production
