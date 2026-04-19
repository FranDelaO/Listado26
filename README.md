# 🎵 Setlist — Gestor de canciones para banda

Aplicación web ligera para que una banda lleve el control de sus canciones: día de toque, día de ensayo, cantante que la interpreta, letra y URL de reproducción. Los datos se sincronizan en tiempo real entre todos los miembros mediante Firebase.

**Modelo de permisos:** una sola persona edita (tú, el admin), el resto solo ve.

---

## 📋 Guía de setup

Son 4 pasos grandes. Calcula unos 15-20 minutos la primera vez.

### Paso 1 — Crear el proyecto de Firebase

1. Entra a [https://console.firebase.google.com](https://console.firebase.google.com) con tu cuenta de Google
2. Clic en **Agregar proyecto**
3. Ponle un nombre (ej: `banda-setlist`)
4. Cuando pregunte por Google Analytics, puedes desactivarlo (no lo necesitas)
5. Espera a que termine de crear el proyecto y clic en **Continuar**

### Paso 2 — Habilitar Firestore y Authentication

**Firestore** (la base de datos):

1. En el menú lateral izquierdo, clic en **Build → Firestore Database**
2. Clic en **Crear base de datos**
3. Elige **Iniciar en modo de producción** (más seguro)
4. Selecciona una ubicación cercana (ej: `us-central` o `southamerica-east1`)
5. Clic en **Habilitar**

Una vez creada, ve a la pestaña **Reglas** (Rules) y reemplaza el contenido con esto:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /songs/{songId} {
      // Cualquiera puede leer (la banda entera)
      allow read: if true;
      // Solo TÚ puedes escribir, modificar o eliminar
      allow write: if request.auth != null
                   && request.auth.token.email == "TU_EMAIL@gmail.com";
    }
  }
}
```

**⚠️ Importante:** reemplaza `TU_EMAIL@gmail.com` por el correo de Google con el que vas a iniciar sesión. Clic en **Publicar**.

**Authentication** (inicio de sesión):

1. En el menú lateral, clic en **Build → Authentication**
2. Clic en **Comenzar**
3. En la pestaña **Sign-in method**, habilita **Google**
4. Elige un nombre público para el proyecto y tu correo de soporte
5. Guarda

### Paso 3 — Obtener la configuración y pegarla en el HTML

1. En Firebase Console, clic en el ícono de engrane ⚙️ junto a "Project Overview" → **Configuración del proyecto**
2. Baja hasta la sección **Tus apps** y clic en el ícono web `</>`
3. Dale un apodo a la app (ej: `setlist-web`) y clic en **Registrar app**
4. Verás un bloque de código con `const firebaseConfig = { ... }`. Cópialo completo.

Ahora abre `index.html` en un editor de texto y busca esta sección al inicio del `<script>`:

```javascript
const firebaseConfig = {
  apiKey: "TU_API_KEY_AQUI",
  authDomain: "tu-proyecto.firebaseapp.com",
  // ...
};

const ADMIN_EMAIL = "tu-email@gmail.com";
```

**Reemplaza**:
- Todo el objeto `firebaseConfig` con el que te dio Firebase
- `ADMIN_EMAIL` con tu correo (el mismo que pusiste en las reglas de Firestore)

Guarda el archivo.

### Paso 4 — Subir a GitHub y activar GitHub Pages

1. Crea una cuenta en [github.com](https://github.com) si no tienes
2. Clic en **New repository** (ícono +)
3. Ponle un nombre (ej: `setlist-banda`), marca **Public**, y clic en **Create repository**
4. Sube los archivos `index.html` y `README.md`:
   - **Opción fácil (arrastrar y soltar):** en la página del repo, clic en **uploading an existing file**, arrastra los archivos, escribe un mensaje de commit ("primer commit") y clic en **Commit changes**
   - **Opción con git** (si ya lo usas): `git add . && git commit -m "primer commit" && git push`
5. En el repo, ve a **Settings → Pages** (menú lateral)
6. En **Source**, elige **Deploy from a branch**, branch `main`, carpeta `/ (root)`, y clic en **Save**
7. Espera unos 1-2 minutos. Arriba aparecerá: *Your site is live at* `https://TU-USUARIO.github.io/setlist-banda/`

**Una cosa importante:** Firebase por defecto solo permite que la app se use desde `localhost`. Tienes que agregar tu dominio de GitHub Pages:

1. En Firebase Console → **Authentication → Settings → Authorized domains**
2. Clic en **Add domain**
3. Pega `TU-USUARIO.github.io` (sin `https://` ni la ruta del repo)
4. Guarda

---

## ✅ Cómo usarlo día a día

### Tú (admin)
1. Abre la URL
2. Clic en **Iniciar sesión** (arriba a la derecha) → elige tu cuenta de Google
3. Aparecerá el botón **+ Nueva Canción** y los íconos de editar/eliminar en cada tarjeta
4. Cualquier cambio se propaga al instante a todos los miembros

### La banda (solo ven)
1. Abre la URL
2. **No necesitan iniciar sesión ni hacer nada** — verán todo el setlist, pueden buscar, filtrar por día o por cantante, ver letras y darle play al URL de reproducción
3. Los cambios que hagas tú aparecen automáticamente sin recargar

---

## 🔒 ¿Es seguro?

- **Sí.** Aunque el código JS es público (está en GitHub), las reglas de Firestore validan en el servidor que solo tu email pueda escribir. Nadie puede hackearlo editando el código del navegador.
- La `apiKey` de Firebase que ves en el HTML **no es un secreto** — Firebase está diseñado para que esa config sea pública. Lo que te protege son las reglas de Firestore y Auth.
- El único cuidado real: **no pierdas acceso a tu cuenta de Google**, porque es la única que puede editar.

---

## 💸 Costos

Gratis para el uso que le va a dar una banda. El *free tier* de Firebase incluye:
- 50,000 lecturas/día y 20,000 escrituras/día en Firestore
- 1 GiB de almacenamiento
- Autenticación ilimitada

Una banda típica no llegará ni al 1% de estos límites.

---

## 🛠️ Solución de problemas

**"Iniciar sesión" no hace nada o da error:**
- Revisa que agregaste `TU-USUARIO.github.io` en los dominios autorizados de Firebase (Paso 4 final)
- Revisa que habilitaste el método de Google en Authentication

**Inicio sesión pero no me aparecen los botones de editar:**
- El email que tienes en `ADMIN_EMAIL` debe ser exactamente igual al correo con el que iniciaste sesión
- Respeta mayúsculas/minúsculas

**Error al guardar "Missing or insufficient permissions":**
- Revisa las reglas de Firestore (Paso 2). El email en las reglas debe coincidir con `ADMIN_EMAIL` del HTML

**No veo el indicador verde "En vivo":**
- Revisa la consola del navegador (F12) para ver el error
- Lo más común: la config de Firebase está mal pegada o falta algún campo

---

## 📁 Estructura de archivos

```
setlist-banda/
├── index.html      # Toda la app (HTML + CSS + JS en un solo archivo)
└── README.md       # Este archivo
```

Sin build, sin dependencias locales, sin servidor. Lo que ves es lo que hay.
