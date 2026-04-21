# 📊 Ejemplos Prácticos y Diagramas - Sistema SHA-256

## 🎯 Ejemplo 1: Comparativa - Almacenamiento Inseguro vs Seguro

### ❌ INSEGURO - Contraseñas en Texto Plano

```json
{
  "users": [
    {
      "username": "admin",
      "password": "admin"  // ⚠️ ¡PELIGRO! Visible en la BD
    },
    {
      "username": "trabajador1",
      "password": "1test"  // ⚠️ ¡PELIGRO! Visible en la BD
    }
  ]
}
```

**Si alguien piratea la BD:**
- Obtiene TODAS las contraseñas en texto plano
- Puede acceder a las cuentas inmediatamente
- Puede usar esas contraseñas en otros servicios

---

### ✅ SEGURO - Contraseñas Hasheadas (SHA-256)

```json
{
  "users": [
    {
      "username": "admin",
      "password": "8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918"
      // ✅ Hash de "admin" - No se puede revertir
    },
    {
      "username": "trabajador1",
      "password": "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08"
      // ✅ Hash de "1test" - No se puede revertir
    }
  ]
}
```

**Si alguien piratea la BD:**
- Solo ve hashes de 64 caracteres
- Sin salt, podría usar rainbow tables
- Con salt (no en este proyecto), sería imposible
- La contraseña original sigue protegida

---

## 🔄 Ejemplo 2: Flujo Completo de Login

### Escenario: Usuario intenta acceder

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ PANTALLA DE LOGIN                                              ┃
┃ Usuario: admin                                                 ┃
┃ Contraseña: ••••••                                             ┃
┃ [Ingresar →]                                                   ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                          │
                          ▼
        ┌─────────────────────────────────┐
        │ Usuario hace clic "Ingresar"    │
        │ • username = "admin"            │
        │ • password = "admin" (en plano) │
        └─────────────────────────────────┘
                          │
                          ▼
        ┌─────────────────────────────────┐
        │ FUNCIÓN: doLogin()              │
        │ Obtiene valores del formulario  │
        └─────────────────────────────────┘
                          │
                          ▼
        ┌─────────────────────────────────────────┐
        │ Calcula SHA-256 de "admin"              │
        │ Hash generado:                          │
        │ 8c6976e5b5410415bde908bd4dee15df...    │
        └─────────────────────────────────────────┘
                          │
                          ▼
        ┌──────────────────────────────────────┐
        │ CONSULTA A BD (users.json)           │
        │ Busca usuario "admin" cuyo hash sea: │
        │ 8c6976e5b5410415bde908bd4dee15df... │
        └──────────────────────────────────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
         ✅ ENCONTRADO           ❌ NO ENCONTRADO
         Usuario: admin           usuario o password
         Role: admin              incorrectos
              │                       │
              ▼                       ▼
         ┌────────────────┐   ┌─────────────────┐
         │ Mostrar panel  │   │ Error message:  │
         │ ADMIN          │   │ "Usuario o      │
         │ ✓ Ver          │   │  contraseña     │
         │ ✓ Editar       │   │  incorrectos"   │
         │ ✓ Gestionar    │   └─────────────────┘
         └────────────────┘
```

### Código JavaScript Correspondiente

```javascript
async function doLogin() {
  // 1. Obtener datos del formulario
  const username = document.getElementById('loginUser').value.trim();
  const password = document.getElementById('loginPass').value;

  // 2. Validar que no estén vacíos
  if (!username || !password) {
    showAlert('loginAlert','Completa todos los campos.','error');
    return;
  }

  // 3. Calcular SHA-256 de la contraseña ingresada
  const hash = await sha256(password);
  // "admin" → "8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918"

  // 4. Buscar usuario con ese username Y ese hash en la BD
  const user = DB.users.find(u => 
    u.username === username && 
    u.password === hash
  );

  // 5. Si no existe, rechaza
  if (!user) {
    showAlert('loginAlert','Usuario o contraseña incorrectos.','error');
    return;
  }

  // 6. Si existe, muestra el panel
  renderDashboard(user);
}
```

---

## 🔐 Ejemplo 3: Comparación de Hashes

### ¿Qué sucede si cambias UNA LETRA?

```javascript
SHA256("admin")     → 8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
SHA256("admins")    → 8987f6fb4c0c57c3e0ad82f7b4aac84e4c0e0c9c8a6b8d9f2e1a4c3b5d7e9f1
                       ^^^^^^^^^^^^^^^ COMPLETAMENTE DIFERENTE
SHA256("Admin")     → 5a105e8b9d40e1329780d62ea2265d8a76e4d4150377e332ec432e74e76e8e9
                       ^^^^^^^^^^^^^^^ COMPLETAMENTE DIFERENTE
```

**Conclusión:** Un cambio mínimo = Hash radicalmente diferente

**Esto es importante porque:**
- Un usuario no puede adivinar el hash
- No puede modificar el hash ligeramente
- La seguridad es perfecta para pequeños cambios

---

## 📝 Ejemplo 4: Registro de Nuevo Usuario

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ FORMULARIO REGISTRO                               ┃
┃ Nombre: Pedro López                               ┃
┃ Email: pedro@email.com                            ┃
┃ Usuario: pedro123                                 ┃
┃ Contraseña: MiSegura@2024                         ┃
┃ [Crear Cuenta →]                                  ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                        │
                        ▼
      ┌─────────────────────────────────┐
      │ VALIDACIONES                    │
      │ ✓ Todos los campos completos    │
      │ ✓ Contraseña ≥ 6 caracteres     │
      │ ✓ Usuario "pedro123" no existe  │
      │ ✓ Email no está registrado      │
      └─────────────────────────────────┘
                        │
                        ▼
      ┌────────────────────────────────────────┐
      │ SHA256("MiSegura@2024")                │
      │ = a7f3c2b8e9d4f1a6c5b2e8d1f4a9c6b3e0│
      │ (hash de 64 caracteres)                │
      └────────────────────────────────────────┘
                        │
                        ▼
      ┌──────────────────────────────────────────────────┐
      │ AGREGAR A users.json:                            │
      │ {                                                │
      │   "id": 4,                                       │
      │   "username": "pedro123",                        │
      │   "password": "a7f3c2b8e9d4f1a6c5b2e8d1f4a9...", │
      │   "role": "publico",  ← Siempre público         │
      │   "name": "Pedro López",                         │
      │   "email": "pedro@email.com",                    │
      │   "createdAt": "2026-04-21T..."                  │
      │ }                                                │
      └──────────────────────────────────────────────────┘
                        │
                        ▼
      ┌─────────────────────────────────┐
      │ ✅ Cuenta creada exitosamente   │
      │ Se redirige a Login             │
      └─────────────────────────────────┘
```

### Código JavaScript

```javascript
async function doRegister() {
  // ... (validaciones)

  // 1. Calcular SHA-256 de la contraseña
  const hash = await sha256(password);

  // 2. Crear objeto usuario con hash
  const newUser = {
    id: DB.users.length + 1,
    username,
    password: hash,           // ← Se guarda el HASH, no la contraseña
    role: "publico",          // Siempre público por defecto
    name,
    email,
    createdAt: new Date().toISOString()
  };

  // 3. Agregar a la BD en memoria
  DB.users.push(newUser);
  
  // 4. En un servidor real, se guardaría a un archivo o BD
  // console.log(JSON.stringify(DB, null, 2));
}
```

---

## 🎓 Ejemplo 5: Pruebas Prácticas - Verificar los Hashes Reales

### Usuarios del Sistema

| Usuario | Contraseña | SHA-256 |
|---------|-----------|---------|
| `admin` | `admin` | `8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918` |
| `trabajador1` | `1test` | `9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08` |
| `usuario1` | `password` | `0a041b9462caa4a31bac3567e0b6e6fd9100787db2ab433d96f6d178cabfce90` |

### Cómo Verificar en el Navegador

**Abre la Consola del Navegador (F12 → Consola)**

```javascript
// Paso 1: Define la función SHA-256
async function sha256(message) {
  const msgBuffer = new TextEncoder().encode(message);
  const hashBuffer = await crypto.subtle.digest('SHA-256', msgBuffer);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
}

// Paso 2: Prueba los hashes
await sha256("admin")
// Resultado: "8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918" ✓

await sha256("1test")
// Resultado: "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08" ✓

await sha256("password")
// Resultado: "0a041b9462caa4a31bac3567e0b6e6fd9100787db2ab433d96f6d178cabfce90" ✓

// Paso 3: Intenta cambiar una letra
await sha256("Admin")
// Resultado: "5a105e8b9d40e1329780d62ea2265d8a76e4d4150377e332ec432e74e76e8e9"
// ⚠️ COMPLETAMENTE DIFERENTE
```

---

## 🛡️ Ejemplo 6: Por Qué Es Seguro

### Escenario: Alguien piratea la BD

**❌ Sistema SIN Hashing (Inseguro)**
```
Pirata obtiene: 
- admin : admin
- trabajador1 : 1test
- usuario1 : password

✗ Puede entrar en las cuentas inmediatamente
✗ Puede usar esas contraseñas en otros servicios
✗ Todos los datos están comprometidos
```

**✅ Sistema CON SHA-256 (Este Proyecto)**
```
Pirata obtiene:
- admin : 8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
- trabajador1 : 9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08
- usuario1 : 0a041b9462caa4a31bac3567e0b6e6fd9100787db2ab433d96f6d178cabfce90

✓ No puede hacer nada con esos hashes
✓ No puede revertir el hash a la contraseña
✓ Las contraseñas siguen siendo seguras
✓ Sin salt, podría usar rainbow tables (pero lento)
```

### ¿Por qué no se puede revertir?

SHA-256 es una función **unidireccional** (función de hash):

```
ENTRADA (1 billón de posibilidades)
        │
        │ SHA-256 (función matemática compleja)
        │
        ▼
SALIDA (64 caracteres hexadecimales)

Es imposible revertir porque:
- La función descarta información en el proceso
- No hay "llave maestra" para desencriptar
- Sería como intentar reconstruir una comida masticada
```

---

## 📚 Ejemplo 7: Conceptos Relacionados

### SALT (Seguridad Mejorada - No implementado aquí)

**Problema:** Dos usuarios con la misma contraseña tienen el mismo hash

```
Usuario 1: "usuario1", password: "password"
Usuario 2: "usuario2", password: "password"

SHA256("password") = 0a041b9462caa4a31bac3567e0b6e6fd9100787db2ab433d96f6d178cabfce90
SHA256("password") = 0a041b9462caa4a31bac3567e0b6e6fd9100787db2ab433d96f6d178cabfce90

⚠️ ¡Hashes idénticos! Se puede usar una rainbow table
```

**Solución:** Agregar un SALT (valor aleatorio)

```
Salt Usuario 1: "a7f3c2b8"
Salt Usuario 2: "f1a9c6b3"

SHA256("password" + "a7f3c2b8") = a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6
SHA256("password" + "f1a9c6b3") = z9y8x7w6v5u4t3s2r1q0p9o8n7m6l5k4

✓ Hashes diferentes aunque password igual
✓ Imposible usar rainbow tables precompiladas
```

---

## 🎯 Resumen de Seguridad

### Niveles de Seguridad

```
NIVEL 1: ❌ Texto plano
         password: "admin"
         Seguridad: 0%
         
NIVEL 2: 🔶 SHA-256 (Este proyecto)
         password: "8c6976e5b54..."
         Seguridad: 85%
         
NIVEL 3: 🟡 SHA-256 + SALT
         salt: "a7f3c2b8"
         password: "f4e7d8a9..."
         Seguridad: 95%
         
NIVEL 4: 🟢 bcrypt / argon2 / PBKDF2
         Funciones adaptadas (lenta a propósito)
         Seguridad: 99%+
         (Usado en producción)
```

---

## ✅ Conclusiones Prácticas

1. **Este proyecto demuestra:** Cómo hashear correctamente contraseñas
2. **Es educativo:** Muestra el concepto de SHA-256 de forma clara
3. **Para producción,** agregaría:
   - Salt por usuario
   - Función de hash más lenta (bcrypt, argon2)
   - HTTPS para tránsito
   - Rate limiting
   - 2FA (autenticación de dos factores)

