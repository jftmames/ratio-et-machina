# Ratio & Machina — Guía de Despliegue

## Arquitectura

```
GitHub Pages (estático)          Supabase (backend)
┌─────────────────────┐         ┌──────────────────────┐
│  index.html          │ ──────▶│  PostgreSQL           │
│  (HTML/CSS/JS)       │        │  ├── articles         │
│                      │        │  ├── submissions      │
│  Supabase JS Client  │        │  ├── reviews          │
│  (CDN)               │        │  ├── board_members    │
└─────────────────────┘         │  Storage (PDFs)       │
                                │  Auth (editores)      │
                                └──────────────────────┘
```

---

## Paso 1: Configurar Supabase (5 min)

1. Ve a **https://supabase.com** y crea una cuenta gratuita
2. Crea un nuevo proyecto (elige la región más cercana: `eu-west-1` para España)
3. Anota las credenciales (Settings → API):
   - **Project URL**: `https://xxxxx.supabase.co`
   - **anon public key**: `eyJhbG...` (la clave pública, NO la service key)

4. Ve a **SQL Editor** → New Query → pega todo el contenido de `supabase-schema.sql` → Run
   - Esto crea las tablas, índices, políticas RLS, bucket de storage y datos iniciales

5. Crea tu usuario editor:
   - Ve a **Authentication** → Users → Add User
   - Email: tu correo
   - Password: elige una contraseña segura
   - Marca "Auto Confirm User"

---

## Paso 2: Configurar el HTML (1 min)

Abre `index.html` y busca estas líneas al inicio del `<script>`:

```javascript
const SUPABASE_URL = 'https://TU-PROYECTO.supabase.co';
const SUPABASE_ANON_KEY = 'TU-ANON-KEY-AQUÍ';
```

Reemplázalas con tus credenciales reales de Supabase.

---

## Paso 3: Desplegar en GitHub Pages (3 min)

### Opción A: Desde la web de GitHub

1. Ve a **https://github.com/new** y crea un repositorio `ratio-et-machina`
2. Sube `index.html` (Add file → Upload files)
3. Ve a **Settings → Pages**
4. Source: Deploy from a branch → `main` → `/ (root)` → Save
5. En 1-2 minutos estará en: `https://tu-usuario.github.io/ratio-et-machina/`

### Opción B: Desde terminal

```bash
git init ratio-et-machina
cd ratio-et-machina
cp /ruta/a/index.html .
git add .
git commit -m "Lanzamiento de Ratio & Machina"
git remote add origin https://github.com/TU-USUARIO/ratio-et-machina.git
git push -u origin main
```

Luego activa Pages en Settings del repositorio.

---

## Paso 4: Dominio propio (opcional)

1. Compra un dominio (ej: `ratioetmachina.org`)
2. En tu proveedor DNS, añade un registro CNAME:
   ```
   CNAME  www  →  tu-usuario.github.io
   ```
3. En GitHub → Settings → Pages → Custom domain → escribe tu dominio
4. Marca "Enforce HTTPS"

---

## Uso de la Revista

### Funciones públicas (sin login)
- **Catálogo**: Cualquier visitante puede buscar y leer artículos
- **Comité**: Página pública del comité editorial
- **Enviar manuscrito**: Cualquier autor puede enviar un artículo (se guarda en Supabase)

### Funciones de editor (requiere login)
- **Panel de revisión**: Ver todos los envíos, cambiar estados, gestionar el flujo editorial
- **Botón "Acceso editor"** en la navegación para iniciar sesión

### Flujo editorial típico
```
Autor envía manuscrito
       ↓
   [PENDIENTE] → Editor revisa → Desk reject O Asignar revisores
       ↓
   [EN REVISIÓN] → Revisores evalúan
       ↓
   [REVISIÓN MENOR/MAYOR] ← → [ACEPTADO] / [RECHAZADO]
       ↓
   Autor corrige y reenvía
       ↓
   [ACEPTADO] → Editor publica como artículo
```

---

## Administración de Datos

### Publicar un artículo aceptado
Desde Supabase Dashboard → Table Editor → `articles` → Insert Row

### Gestionar el comité editorial
Table Editor → `board_members` → Editar, añadir o eliminar miembros

### Ver manuscritos subidos
Storage → `manuscripts` → Descargar archivos

---

## Límites del Tier Gratuito de Supabase

| Recurso | Límite gratuito |
|---------|----------------|
| Base de datos | 500 MB |
| Storage | 1 GB |
| Ancho de banda | 5 GB/mes |
| Auth usuarios | 50.000 MAU |
| API requests | Ilimitadas |

Para una revista académica, estos límites son más que suficientes.

---

## Estructura de la Base de Datos

| Tabla | Descripción | Acceso público |
|-------|------------|---------------|
| `articles` | Artículos publicados | ✅ Lectura |
| `board_members` | Comité editorial | ✅ Lectura |
| `submissions` | Envíos de manuscritos | ✅ Solo insertar |
| `reviews` | Revisiones por pares | ❌ Solo editores |

---

## Siguientes pasos sugeridos

- [ ] Conectar un servicio de email (Resend, SendGrid) para notificaciones automáticas
- [ ] Añadir generación de DOI real (Crossref, DataCite)
- [ ] Integrar ORCID OAuth para verificar identidad de autores
- [ ] Crear una página de política editorial y ética de publicación
- [ ] Solicitar ISSN al organismo nacional correspondiente
- [ ] Indexar en DOAJ, Dialnet, PhilPapers, Latindex
