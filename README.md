# CuentasRápidas

Sistema para organizar gastos personales con un tablero Kanban visual. Disponible como app web (Angular), app móvil/nativa (Flutter) y con backend en Firebase.

---

## Descripción

CuentasRápidas permite registrar, priorizar y mover gastos entre columnas **Por Pagar → Carrito → Pagado**, con presupuesto por tablero, sistema de prioridades (Vital / Gusto / Capricho) y gráficos de análisis en tiempo real.

---

## Stack

| Capa | Tecnología |
|------|-----------|
| Web | Angular + Angular Material + CDK DragDrop |
| Móvil | Flutter (Android + iOS) |
| Backend | Firebase Firestore + Firebase Auth |
| Auth | Google Sign-In |
| Deploy | Firebase Hosting (web) |

---

## Estructura del repositorio

```
cuentasrapidas/
├── flutter_app/       # App nativa Android + iOS
├── angular_app/       # App web Angular
├── firebase/          # Reglas Firestore, índices, config emuladores
├── shared/            # Constantes y esquemas compartidos
├── planin.md          # Plan de implementación detallado
└── README.md
```

---

## Requisitos previos

- Node.js >= 20
- Angular CLI >= 17 (`npm install -g @angular/cli`)
- Flutter SDK >= 3.22
- Firebase CLI (`npm install -g firebase-tools`)
- Cuenta de Firebase con proyecto creado

---

## Configuración inicial

### 1. Clonar el repositorio

```bash
git clone <repo-url>
cd cuentasrapidas
```

### 2. Configurar Firebase

```bash
firebase login
firebase use --add   # selecciona tu proyecto
```

Copia tus credenciales de Firebase en:
- `angular_app/src/environments/environment.ts`
- `flutter_app/lib/firebase_options.dart` (generado con FlutterFire CLI)

### 3. Deploy de reglas Firestore

```bash
firebase deploy --only firestore:rules
```

---

## Desarrollo local

### Web (Angular)

```bash
cd angular_app
npm install
ng serve
```

App disponible en `http://localhost:4200`.

### Flutter

```bash
cd flutter_app
flutter pub get
flutter run
```

### Emuladores Firebase (recomendado para desarrollo)

```bash
firebase emulators:start
```

Esto levanta Firestore y Auth de forma local sin afectar producción.

---

## Variables de entorno

Nunca subas credenciales reales al repositorio. Usa el archivo `environment.ts` (ignorado por `.gitignore`) para la config de Firebase:

```typescript
// angular_app/src/environments/environment.ts
export const environment = {
  production: false,
  firebase: {
    apiKey: 'TU_API_KEY',
    authDomain: 'TU_PROYECTO.firebaseapp.com',
    projectId: 'TU_PROYECTO',
    storageBucket: 'TU_PROYECTO.appspot.com',
    messagingSenderId: 'TU_SENDER_ID',
    appId: 'TU_APP_ID',
  }
};
```

---

## Schema de datos (Firestore)

```
/users/{uid}/
  boards/{boardId}/
    name: string
    createdAt: timestamp
    expenses/{expenseId}/
      titulo:    string
      monto:     number          # pesos chilenos, sin decimales
      estado:    "porPagar" | "carrito" | "pagado"
      prioridad: "vital" | "gusto" | "capricho"
      createdAt: timestamp
  budgets/{boardId}/
    amount: number
```

---

## Reglas de seguridad

Solo el usuario autenticado puede leer y escribir sus propios datos:

```
match /users/{uid}/{document=**} {
  allow read, write: if request.auth != null && request.auth.uid == uid;
}
```

---

## Deploy a producción

```bash
# Build Angular
cd angular_app
ng build --configuration production

# Deploy hosting + rules
firebase deploy
```

---

## Reglas de negocio

- El monto se ingresa en **pesos chilenos** y no acepta decimales.
- Las columnas tienen orden estricto: **Por Pagar → Carrito → Pagado**.
- Cada tablero tiene un presupuesto opcional asociado a la columna Carrito.
- Las prioridades son: **Vital** (no negociable), **Gusto** (parte del estilo de vida), **Capricho** (prescindible).

---

## Contribuir

1. Crea un branch desde `main`: `git checkout -b feature/nombre`
2. Haz tus cambios con commits atómicos y descriptivos.
3. Abre un Pull Request con descripción del cambio y cómo probarlo.
4. Asegúrate de que los emuladores pasen antes de hacer merge.

---

## Licencia

MIT
