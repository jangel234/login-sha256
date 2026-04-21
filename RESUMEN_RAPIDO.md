# ⚡ Resumen Rápido - SHA-256 + SALT

## 🎯 En 1 Minuto

```
¿Qué es? → SHA-256 (función hash) + SALT (valor aleatorio)
Resultado → Dos capas de seguridad
¿Reversible? → NO (imposible volver atrás)
Seguridad → ✓ Rainbow tables inútiles
           ✓ Imposible crackear en tiempo práctico
```

---

## 📊 Tabla Rápida

| Concepto | Sin SALT | Con SALT |
|----------|----------|----------|
| **¿Dos usuarios, misma password?** | Hashes iguales ✗ | Hashes diferentes ✓ |
| **¿Rainbow table efectiva?** | Sí ✗ | No ✓ |
| **Seguridad** | Media 🟡 | Alta 🟢 |
| **Estándar** | No ✗ | Sí ✓ |

---

## 🔄 Flujo Mínimo

### Registro:
```
Password: "abc123"
     ↓
Generar SALT: "a7f3c2b8e9d4f1a6"
     ↓
SHA256("abc123" + "a7f3c2b8e9d4f1a6")
     ↓
Hash: "xyz789abc..."
     ↓
Guardar BD: {salt, hash}
```

### Login:
```
Password: "abc123"
     ↓
Recuperar SALT de BD: "a7f3c2b8e9d4f1a6"
     ↓
SHA256("abc123" + "a7f3c2b8e9d4f1a6")
     ↓
¿Coincide hash? → SÍ/NO
```

---

## 💾 Base de Datos (Mejorada)

```json
{
  "username": "admin",
  "salt": "a7f3c2b8e9d4f1a6",
  "password": "f3a4b2c8d9e1f3a4b2c8d9e1f3a4b2c8d9e1f3..."
}
```

---

## 🧮 Ejemplos

| Contraseña | SALT | SHA-256 + SALT |
|-----------|------|----------------|
| admin | a7f3c2b8e9d4f1a6 | f3a4b2c8d9e1... |
| 1test | b5e1c3d2a7f8c9b1 | c7d8e9f1a2b3... |
| password | f2a8b1c3d5e7f9a2 | d4e5f6a7b8c9... |

---

## 🔐 Por Qué Es Más Seguro

```
SIN SALT:
"password" → SHA256 → abc123...
Si hackean: abc123... → rainbow table → password ✗

CON SALT:
"password" + SALT → SHA256 → xyz789...
Si hackean: xyz789... → rainbow table → ??? (SALT es único) ✓
```

---

## 📱 Código Esencial

```javascript
// Generar SALT
function generateSalt() {
  const bytes = crypto.getRandomValues(new Uint8Array(8));
  return Array.from(bytes)
    .map(b => b.toString(16).padStart(2, '0'))
    .join('');
}

// SHA-256 + SALT
async function sha256WithSalt(password, salt) {
  return await sha256(password + salt);
}

// En Registro
const salt = generateSalt();
const hash = await sha256WithSalt(password, salt);
addUser({ username, password: hash, salt });

// En Login
const user = DB.users.find(u => u.username === username);
const hash = await sha256WithSalt(password, user.salt);
if (hash === user.password) { /* acceso */ }
```

---

## ✅ Checklist - Lo Que Debe Saber

- [ ] SALT es valor aleatorio único por usuario
- [ ] SHA-256 + SALT es más seguro que solo SHA-256
- [ ] Dos usuarios con misma contraseña = hashes diferentes
- [ ] Se guarda el SALT junto con el hash
- [ ] Rainbow tables son inútiles con SALT
- [ ] El login recupera el SALT del usuario
- [ ] Se concatena contraseña + SALT antes de hashear
- [ ] Es estándar en todas las BD profesionales

---

## 🎓 Respuestas de 10 Segundos

**P: ¿Qué es SALT?**
R: Valor aleatorio único que se agrega a la contraseña

**P: ¿Por qué se usa?**
R: Evita rainbow tables y hace único el hash de cada usuario

**P: ¿Se guarda el SALT?**
R: Sí, es público. No necesita encriptación

**P: ¿Cómo se genera?**
R: Con `crypto.getRandomValues()` (aleatorio criptográfico)

**P: ¿Mejora mucho la seguridad?**
R: Sí, de 5/10 a 8/10 automáticamente

---

## 🎬 Demo en 30 Segundos

1. Abre console (F12)
2. Copia función sha256()
3. `await sha256("admin" + "a7f3c2b8e9d4f1a6")` → muestra hash
4. `await sha256("admin")` → muestra diferente
5. "Así funciona con SALT vs sin SALT"

---

## 📐 Diagramas Mínimos

### Almacenamiento
```
Antes:
username: admin
password: 8c6976e5...

Ahora:
username: admin
salt: a7f3c2b8e9d4f1a6
password: f3a4b2c8d9e1...
```

### Proceso
```
Entrada: "admin"
   ↓
+ SALT: "a7f3c2b8e9d4f1a6"
   ↓
SHA-256
   ↓
Resultado: f3a4b2c8d9e1...
```

---

## 🚀 Jerarquía de Seguridad

1. ❌ Texto plano → `password: "admin"`
2. 🟡 SHA-256 → `password: "8c6976e5..."`
3. 🟢 SHA-256 + SALT → `salt: "a7f3c...", password: "f3a4b..."`
4. 🟢🟢 bcrypt/argon2 → `$2b$12$...`

Este proyecto está en nivel 3/4 (profesional)

---

## 🎯 Punto Principal

> "SALT hace que cada usuario tenga un hash diferente
> aunque la contraseña sea igual, 
> imposibilitando ataques con tablas precompiladas"



