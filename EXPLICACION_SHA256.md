# 🔐 Sistema de Hasheo SHA-256 para Autenticación

## 📋 Índice
1. [¿Qué es el Hasheo?](#qué-es-el-hasheo)
2. [¿Qué es SHA-256?](#qué-es-sha-256)
3. [Cómo Funciona en Este Proyecto](#cómo-funciona-en-este-proyecto)
4. [Implementación en el Código](#implementación-en-el-código)
5. [Flujo de Autenticación](#flujo-de-autenticación)
6. [Ventajas y Seguridad](#ventajas-y-seguridad)

---

## ¿Qué es el Hasheo?

El **hasheo** es una función criptográfica que:
- Convierte una cadena de texto de cualquier longitud en una cadena de caracteres de **longitud fija**
- Es **unidireccional**: no se puede reversar para obtener la contraseña original
- Es **determinista**: la misma entrada siempre genera el mismo hash
- Es **sensible**: cambiar un carácter genera un hash completamente diferente

### Ejemplo:
```
Entrada: "admin" → Hash: 8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
Entrada: "admi"  → Hash: e3ad214e56ab63d0a9526b1245215f566d78f3427afc4df8db25795d9d4e57f6
(Cambiar 1 carácter = Hash completamente diferente)
```

**¿Por qué es importante?**
- La contraseña NO se almacena en la base de datos
- Se almacena solo el hash
- Incluso si alguien accede a la base de datos, no puede recuperar la contraseña original

---

## ¿Qué es SHA-256?

**SHA-256** (Secure Hash Algorithm - 256 bits) es:
- Un algoritmo de **hashing criptográfico** estándar
- Produce hashes de **256 bits** (64 caracteres hexadecimales)
- Es parte de la familia SHA-2
- Considerado **seguro** para la mayoría de aplicaciones
- Usado por Bitcoin, SSL/TLS y muchos sistemas de seguridad

### Características:
- **Longitud fija**: Siempre produce 64 caracteres
- **Velocidad consistente**: Tarda ~0.1ms en computadoras modernas
- **Unidireccional**: Imposible revertir el proceso
- **Colisión rara**: Prácticamente imposible encontrar dos entradas con mismo hash

### Formato del Hash SHA-256:
```
8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
└─ 64 caracteres hexadecimales (números 0-9 y letras a-f)
└─ Representan 256 bits de información
```

---

## Cómo Funciona en Este Proyecto

### 1. **Base de Datos (users.json)**
Las contraseñas se almacenan como hashes SHA-256:

```json
{
  "id": 1,
  "username": "admin",
  "password": "8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918",
  "role": "admin",
  "name": "Administrador"
}
```

**El archivo `credenciales.txt` muestra las contraseñas ORIGINALES (solo con fines educativos):**
```
Usuario    Contraseña  Rol
admin      admin       Admin
trabajador1 1test      Trabajador
usuario1   password    Público
```

### 2. **Conversión Hash**
Cada contraseña original se convierte a SHA-256:

| Usuario | Contraseña | SHA-256 |
|---------|-----------|---------|
| admin | admin | 8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918 |
| trabajador1 | 1test | 9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08 |
| usuario1 | password | 0a041b9462caa4a31bac3567e0b6e6fd9100787db2ab433d96f6d178cabfce90 |

---

## Implementación en el Código

### Función SHA-256 en JavaScript:

```javascript
// Función para calcular SHA-256
async function sha256(message) {
  // 1. Convertir texto a bytes (codificación UTF-8)
  const msgBuffer = new TextEncoder().encode(message);
  
  // 2. Aplicar el algoritmo SHA-256 (usa la API de criptografía del navegador)
  const hashBuffer = await crypto.subtle.digest('SHA-256', msgBuffer);
  
  // 3. Convertir bytes a formato hexadecimal
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  
  // 4. Convertir cada byte a hex (2 dígitos) y unir
  return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
}
```

### Desglose Paso a Paso:

**Paso 1: Codificación UTF-8**
```javascript
"admin" → [97, 100, 109, 105, 110] (bytes)
```

**Paso 2: Aplicar SHA-256**
- El algoritmo procesa los bytes
- Realiza operaciones matemáticas complejas
- Produce 32 bytes (256 bits)

**Paso 3: Conversión a Hexadecimal**
```javascript
[140, 105, 118, 229, 181, 65, 4, 21, ...] 
→ ["8c", "69", "76", "e5", "b5", "41", "04", "15", ...]
```

**Paso 4: Unir en String**
```javascript
"8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918"
```

---

## Flujo de Autenticación

### 🔵 Inicio de Sesión (Login)

```
┌─────────────────────────────────┐
│ Usuario ingresa contraseña      │
│ Ejemplo: "admin"                │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│ Sistema calcula SHA-256         │
│ "admin" → 8c6976e5...           │
└──────────────┬──────────────────┘
               │
               ▼
┌─────────────────────────────────┐
│ Compara con hash en BD          │
│ ¿8c6976e5... == 8c6976e5... ?   │
└──────────────┬──────────────────┘
               │
    ┌──────────┴──────────┐
    ▼                     ▼
  ✅ MATCH            ❌ NO MATCH
  Acceso              Rechaza
  Concedido           Acceso
```

### 🟢 Registro de Nuevo Usuario (Register)

```
┌────────────────────────────────┐
│ Usuario ingresa contraseña:    │
│ "miContraseña123"              │
└──────────────┬─────────────────┘
               │
               ▼
┌────────────────────────────────┐
│ Sistema valida:                │
│ • Mínimo 6 caracteres ✓        │
│ • Usuario no existe ✓          │
│ • Email no registrado ✓        │
└──────────────┬─────────────────┘
               │
               ▼
┌────────────────────────────────┐
│ Calcula SHA-256 de la pass     │
│ "miContraseña123" → 2a5e...    │
└──────────────┬─────────────────┘
               │
               ▼
┌────────────────────────────────┐
│ Guarda en users.json:          │
│ {                              │
│   username: "nuevo_user",      │
│   password: "2a5e...",         │
│   role: "publico"              │
│ }                              │
└──────────────┬─────────────────┘
               │
               ▼
┌────────────────────────────────┐
│ ✅ Usuario creado              │
└────────────────────────────────┘
```

---

## Ventajas y Seguridad

### ✅ Ventajas de SHA-256

| Aspecto | Beneficio |
|--------|-----------|
| **Irreversible** | No se puede obtener la contraseña del hash |
| **Determinista** | Misma contraseña = Mismo hash siempre |
| **Rápido** | Calcula en microsegundos |
| **Sensible** | Un carácter diferente = Hash completamente diferente |
| **Estándar** | Ampliamente aceptado y estudiado |
| **Distribuido** | Usado en Bitcoin, SSL, sistemas financieros |

### 🔒 Seguridad en Este Sistema

**Lo que protege:**
- ✅ Las contraseñas NO están en la base de datos
- ✅ Si alguien accede a users.json, solo ve hashes
- ✅ No puede usar un hash para iniciar sesión (debo hashear nuevamente)
- ✅ Se valida en cada login que el hash coincida

**Limitaciones (para mejorar):**
- ⚠️ Sin SALT: Se podría usar "rainbow tables" (tablas precalculadas)
- ⚠️ Sin rate limiting: Alguien podría probar muchas contraseñas
- ⚠️ En cliente: Idealmente se hashea en servidor (aquí es demostración)
- ⚠️ Sin HTTPS: En internet se debe usar conexión encriptada

---

## Cómo Verificar el Hash Manualmente

### Online
1. Ir a: https://www.sha256online.com/
2. Escribir: `admin`
3. Resultado: `8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918` ✅

### En Navegador (DevTools - Consola)
```javascript
// Copiar esta función en la consola
async function sha256(message) {
  const msgBuffer = new TextEncoder().encode(message);
  const hashBuffer = await crypto.subtle.digest('SHA-256', msgBuffer);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
}

// Usar
await sha256("admin")
// Resultado: "8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918"
```

---

## Resumen de Conceptos Clave

| Concepto | Explicación |
|----------|------------|
| **Hash** | Función que convierte datos en cadena fija irreversible |
| **SHA-256** | Algoritmo estándar que produce hashes de 256 bits |
| **Contraseña** | Lo que el usuario ingresa (NUNCA se guarda) |
| **Hash SHA-256** | Lo que se guarda en la BD (imposible revertir) |
| **Autenticación** | Comparar: SHA256(entrada) == SHA256(guardado) |
| **Colisión** | Dos entradas con mismo hash (prácticamente imposible en SHA-256) |
| **Rainbow Table** | Tabla precalculada de hashes (se evita con SALT) |

---

## Conclusión

Este sistema demuestra que:
1. **Las contraseñas NO deben guardarse** en texto plano
2. **SHA-256 es rápido y seguro** para hashear
3. **La autenticación funciona comparando hashes**, no comparando contraseñas
4. **Los hashes son unidireccionales**: si se pierde la BD, las contraseñas están a salvo

🎓 **Para una aplicación en producción**, deberías agregar:
- Salt (valor aleatorio por usuario)
- Key derivation (PBKDF2, bcrypt, argon2)
- HTTPS (encriptación en tránsito)
- Rate limiting (limitar intentos de acceso)

