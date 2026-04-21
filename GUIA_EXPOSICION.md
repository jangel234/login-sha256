# 🎓 Guía de Exposición - Sistema SHA-256

## 📌 Estructura Recomendada para tu Presentación

---

## Parte 1: Introducción (2-3 minutos)

### Abre con una pregunta:
> "¿Saben ustedes dónde se guardan las contraseñas en las aplicaciones web? ¿Las guardan en texto plano como `usuario: admin, contraseña: admin`?"

**La respuesta: NO, eso sería una pesadilla de seguridad.**

### Contexto del Proyecto:
- Sistema de login educativo
- Implementa autenticación SHA-256
- Simula una base de datos real
- Demuestra buenas prácticas de seguridad

---

## Parte 2: ¿Qué es el Hasheo? (3-4 minutos)

### Concepto Simple:
"Un hash es como un **impresor de huellas dactilares para datos**."

### Características Clave (Mostrar en pantalla):

```
ENTRADA (cualquier longitud)
    ↓
FUNCIÓN CRIPTOGRÁFICA
    ↓
SALIDA FIJA (64 caracteres en SHA-256)
    ↓
UNIDIRECCIONAL (no se puede revertir)
```

### Ejemplo Visual:

```
"abc"        →  [SHA-256]  → ba7816bf8f...
"abcd"       →  [SHA-256]  → 88d4266fd4...
"abcde"      →  [SHA-256]  → 36a9e7f1c9...
"admin"      →  [SHA-256]  → 8c6976e5b5...
"ADMIN"      →  [SHA-256]  → 5a105e8b9d...
```

**Conclusión:** Un pequeño cambio = Resultado completamente diferente

---

## Parte 3: ¿Qué es SHA-256? (2-3 minutos)

### Explicación Técnica:
- **SHA** = Secure Hash Algorithm (Algoritmo de Hash Seguro)
- **256** = Produce 256 bits de salida
- **Estándar** = Usado por Bitcoin, SSL/TLS, gobiernos

### Características:
```
┌─────────────────────────────────────┐
│ SHA-256 (Secure Hash Algorithm 256) │
├─────────────────────────────────────┤
│ ✓ Velocidad: ~0.1ms (muy rápido)    │
│ ✓ Irreversible: 100% imposible      │
│ ✓ Determinista: Mismo resultado     │
│ ✓ Unidireccional: No se revierte    │
│ ✓ Sensible: Un bit = Todo diferente │
└─────────────────────────────────────┘
```

### Tamaño del Hash:
```
256 bits = 32 bytes = 64 caracteres hexadecimales

8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
↑ 64 caracteres, cada par = 1 byte
```

---

## Parte 4: Cómo Funciona en Este Proyecto (5-7 minutos)

### Arquitectura:

```
┌────────────────┐
│  index.html    │
│  (Navegador)   │
└───────┬────────┘
        │
        ├─ JavaScript: Función sha256()
        │              (calcula en cliente)
        │
        └─ Almacena: users.json
                     (datos de usuarios)
```

### Flujo de Login:

**PANTALLA:**
```
Usuario: admin
Contraseña: ••••••
[Ingresar]
```

**PROCESO INTERNO:**
```
1. Usuario escribe: "admin"
   
2. JavaScript calcula:
   sha256("admin") = 8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
   
3. Busca en BD (users.json):
   ¿Existe usuario "admin" con hash 8c6976e5...?
   
4. Resultado:
   ✓ SÍ existe → LOGIN EXITOSO
   ✗ NO existe → ERROR
```

### Base de Datos (users.json):

```json
[
  {
    "username": "admin",
    "password": "8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918",
    "role": "admin"
  },
  {
    "username": "trabajador1",
    "password": "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
    "role": "trabajador"
  }
]
```

**Nota importante:**
> "La contraseña original NUNCA se almacena. Solo el hash."

---

## Parte 5: Mostrar el Código (3-4 minutos)

### La Función SHA-256 (Explicar línea por línea):

```javascript
async function sha256(message) {
  // Paso 1: Convertir texto a bytes (UTF-8)
  const msgBuffer = new TextEncoder().encode(message);
  
  // Paso 2: Aplicar el algoritmo SHA-256
  const hashBuffer = await crypto.subtle.digest('SHA-256', msgBuffer);
  
  // Paso 3: Convertir bytes a números
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  
  // Paso 4: Convertir a hexadecimal (formato visible)
  return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
}
```

### La Función de Login (Explicar):

```javascript
async function doLogin() {
  // 1. Obtener datos del formulario
  const username = document.getElementById('loginUser').value;
  const password = document.getElementById('loginPass').value;
  
  // 2. Calcular SHA-256
  const hash = await sha256(password);
  
  // 3. Buscar en la BD
  const user = DB.users.find(u => 
    u.username === username && 
    u.password === hash
  );
  
  // 4. Si existe, login exitoso
  if (user) {
    renderDashboard(user);
  } else {
    showAlert('Usuario o contraseña incorrectos');
  }
}
```

---

## Parte 6: Demostración Práctica (2-3 minutos)

### Live Demo - Mostrar en vivo:

**OPCIÓN 1: En el Navegador**

```
1. Abre la consola (F12)
2. Copia la función sha256()
3. Ejecuta: await sha256("admin")
4. Muestra el resultado: 8c6976e5b541...
5. Prueba: await sha256("Admin")
   → Resultado diferente
6. Prueba: await sha256("admin ")
   → Resultado diferente
```

**OPCIÓN 2: Mostrar El Sistema Funcionando**

```
1. Abre index.html en navegador
2. Intenta login con: admin / admin
   ✓ SUCCESS
3. Intenta: admin / admins
   ✗ ERROR
4. Intenta: admin / password
   ✗ ERROR
```

---

## Parte 7: Seguridad - Por Qué Funciona (3-4 minutos)

### Problema que Resuelve:

```
❌ INSEGURO (Antes):
   Base de Datos: [admin : admin, trabajador : 1test]
   Si alguien hackea → Obtiene TODAS las contraseñas

✅ SEGURO (Ahora):
   Base de Datos: [admin : 8c6976e5..., trabajador : 9f86d081...]
   Si alguien hackea → Solo obtiene hashes inútiles
```

### ¿Por Qué Es Imposible Revertir?

```
"admin" → SHA-256 → 8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918

¿Se puede ir al revés?
8c6976e5... → ???? → "admin"

NO, porque:
1. El algoritmo descarta información (función de una vía)
2. No hay "llave" para desencriptar
3. Matemáticamente imposible
```

### Comparación: Encriptación vs Hashing

```
ENCRIPTACIÓN (reversible con llave):
"admin" + llave → 8c6976e5...
8c6976e5... + llave → "admin"  ✓ Se puede revertir

HASHING (irreversible):
"admin" → 8c6976e5...
8c6976e5... → ??? ✗ NO se puede revertir
```

---

## Parte 8: Limitaciones y Mejoras (2-3 minutos)

### Limitaciones de Este Sistema:

| Problema | Solución |
|----------|----------|
| Sin SALT | Usar salt por usuario (valor aleatorio) |
| Sin rate limiting | Limitar intentos de login |
| En cliente | En producción, hashear en servidor |
| Sin HTTPS | Encriptación en tránsito |
| SHA-256 rápido | Usar bcrypt/argon2 (más lentas) |

### Mejoras para Producción:

```
NIVEL ACTUAL (Educativo):
SHA-256 sin salt

NIVEL MEJORADO (Medio):
SHA-256 + SALT (valor aleatorio por usuario)

NIVEL PROFESIONAL (Producción):
bcrypt / argon2 / PBKDF2
(Funciones de derivación de claves)
```

---

## Parte 9: Conclusiones (1-2 minutos)

### Puntos Clave:

1. ✅ **Nunca almacenar contraseñas en texto plano**
2. ✅ **SHA-256 es rápido y seguro para hashear**
3. ✅ **El hash es irreversible y unidireccional**
4. ✅ **Autenticación = Comparar hashes, no texto**
5. ✅ **Pequeños cambios = Hashes completamente diferentes**

### Impacto Real:
> "Gracias a funciones como SHA-256, cuando te registras en un sitio web, tu contraseña está protegida incluso si alguien accede a la base de datos."

---

## 📊 Diagramas para Presentación

### Diagrama 1: Flujo de Autenticación

```
┌─────────────────────────────────────────┐
│ USUARIO INTENTA ACCEDER                 │
│ Usuario: admin, Password: admin         │
└──────────────┬──────────────────────────┘
               │
               ▼
┌──────────────────────────────────────┐
│ CALCULAR HASH                        │
│ sha256("admin") = 8c6976e5...        │
└──────────────┬───────────────────────┘
               │
               ▼
┌──────────────────────────────────────────────┐
│ COMPARAR CON BASE DE DATOS                   │
│ ¿user.password == 8c6976e5...?               │
└──────┬───────────────────────────┬───────────┘
       │ SÍ                       │ NO
       ▼                         ▼
    ✓ ACCESO                  ✗ ERROR
    CONCEDIDO                 ACCESO DENEGADO
```

### Diagrama 2: Diferencia de Seguridad

```
SIN HASHING              CON SHA-256 (ESTE PROYECTO)
─────────────────────    ─────────────────────────────
Contraseña: admin        Hash: 8c6976e5b541...
Contraseña: 1test        Hash: 9f86d081884c...
Contraseña: password     Hash: 0a041b9462ca...

Si hackean:              Si hackean:
✗ Ven todas las          ✓ Solo ven números aleatorios
  contraseñas            ✓ No pueden usarlos para entrar
✗ Pueden entrar          ✓ Las contraseñas siguen seguras
  inmediatamente
```

---

## 🎤 Frases Clave para Recordar

- **"SHA-256 es como una huella dactilar digital"**
- **"El hash es un camino de una sola vía"**
- **"La contraseña original NUNCA se almacena"**
- **"Cambiar un carácter = Hash completamente diferente"**
- **"Es matemáticamente imposible revertir"**
- **"La seguridad depende de no poder ver la contraseña"**

---

## ⏱️ Tiempo Estimado Total: 20-25 minutos

```
Introducción............2-3 min
¿Qué es Hasheo?........3-4 min
¿Qué es SHA-256?.......2-3 min
Cómo Funciona..........5-7 min
Mostrar Código.........3-4 min
Demo Práctica..........2-3 min
Seguridad..............3-4 min
Limitaciones...........2-3 min
Conclusiones...........1-2 min
PREGUNTAS..............5-10 min
─────────────────────
TOTAL..................25-35 min
```

---

## 📋 Checklist Antes de Presentar

- [ ] Verificar que index.html abre en navegador
- [ ] Probar login con: admin / admin
- [ ] Tener la consola (F12) lista para mostrar sha256()
- [ ] Tener los archivos abiertos (index.html, users.json)
- [ ] Revisar la conexión a internet (o tener web offline)
- [ ] Probar los diagramas/imágenes
- [ ] Conocer las respuestas a preguntas comunes
- [ ] Tener ejemplos memorizados (admin, trabajador1, usuario1)

---

## 🤔 Preguntas Probables y Respuestas

### P: "¿Qué es exactamente un hash?"
R: "Es una función que convierte cualquier texto en una cadena fija de 64 caracteres. Es como una huella dactilar digital: única y no reversible."

### P: "¿Por qué no encriptamos en lugar de hashear?"
R: "Porque la encriptación es reversible (si tienes la llave). Nosotros NO queremos poder reversar. La contraseña debe ser imposible de recuperar."

### P: "¿Es SHA-256 completamente seguro?"
R: "Es seguro para la mayoría de usos, pero en producción se agregan mejoras como SALT y funciones más lentas (bcrypt)."

### P: "¿Qué pasa si alguien prueba muchas contraseñas?"
R: "Sin rate limiting, alguien podría intentar muchas. En producción se limita a 5 intentos, luego bloquea la cuenta."

### P: "¿Dónde se calcula el hash: en cliente o servidor?"
R: "En este proyecto, en el cliente (navegador). En producción, siempre en el servidor para mayor seguridad."

