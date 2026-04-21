# ⚡ Resumen Rápido - SHA-256

## 🎯 En 1 Minuto

```
¿Qué es? → Función que convierte texto en números aleatorios
Resultado → 64 caracteres iguales siempre
¿Reversible? → NO (imposible volver atrás)
Seguridad → ✓ No se puede saber la contraseña del hash
```

---

## 📊 Tabla Rápida

| Concepto | Explicación | Ejemplo |
|----------|------------|---------|
| **Input** | Texto cualquiera | `"admin"` |
| **SHA-256** | Función matemática | `crypto.subtle.digest()` |
| **Output** | Hash de 64 caracteres | `8c6976e5...` |
| **Propiedad** | Unidireccional | No se puede revertir |
| **Uso** | Almacenar contraseñas | users.json |

---

## 🔄 Flujo Mínimo

```
Usuario digita: "admin"
        ↓
sha256("admin")
        ↓
8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
        ↓
Compara con BD
        ↓
¿Coincide? → SÍ = LOGIN / NO = ERROR
```

---

## 💾 Base de Datos

```
users.json:
{
  "username": "admin",
  "password": "8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918"
}

credenciales.txt (solo referencia):
admin = admin
```

---

## 🧮 Ejemplos Rápidos

| Texto | SHA-256 |
|------|---------|
| admin | 8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918 |
| Admin | 5a105e8b9d40e1329780d62ea2265d8a76e4d4150377e332ec432e74e76e8e9 |
| admin  | (con espacio final) f3751c1f4b16e968b6d85ebec11c2b13ac77fb8bd36c4f17c0e1b2f48cd2e... |

**Conclusión:** Cambiar UNA LETRA = HASH COMPLETAMENTE DIFERENTE

---

## 🔐 Por Qué Es Seguro

```
❌ Texto Plano
   Si hackean → Contraseña visible

✅ SHA-256
   Si hackean → Solo ven hashes
              → No pueden revertir
              → Contraseña sigue segura
```

---

## 📱 Código Esencial

```javascript
// La función
async function sha256(msg) {
  const buf = new TextEncoder().encode(msg);
  const hash = await crypto.subtle.digest('SHA-256', buf);
  const arr = Array.from(new Uint8Array(hash));
  return arr.map(b => b.toString(16).padStart(2, '0')).join('');
}

// El login
const hash = await sha256(passwordIngresada);
const user = BD.find(u => u.password === hash);
if (user) { login exitoso } else { error }
```

---

## ✅ Checklist - Lo Que Debe Saber

- [ ] SHA-256 produce 64 caracteres
- [ ] Es irreversible (no se puede deshacer)
- [ ] Cambia completamente con pequeños cambios
- [ ] Se almacena el hash, no la contraseña
- [ ] El login compara hashes
- [ ] Es seguro porque no se puede reversar
- [ ] Se mejora con SALT en producción
- [ ] Funciona en navegador con `crypto.subtle.digest()`

---

## 🎓 Respuestas de 10 Segundos

**P: ¿Qué es SHA-256?**
R: Función que convierte texto en números fijos de 64 caracteres

**P: ¿Se puede revertir?**
R: No, es matemáticamente imposible

**P: ¿Entonces cómo se loguea?**
R: Calcula el hash nuevamente y compara: ¿hash(entrada) == hash(BD)?

**P: ¿Es seguro?**
R: Sí, porque aunque alguien tenga el hash, no puede obtener la contraseña

**P: ¿Dónde se almacena?**
R: En users.json solo el hash, nunca la contraseña original

---

## 🎬 Demo en 30 Segundos

1. Abre console (F12)
2. Copia función sha256()
3. `await sha256("admin")` → muestra hash
4. `await sha256("Admin")` → muestra diferente
5. Abre users.json → muestra hashes guardados
6. "Eso es todo lo que se almacena, solo números"

---

## 📐 Diagramas Mínimos

### Login
```
Entrada      Procesado    Resultado
"admin"   →  SHA-256   →  8c6976e5...
            
Compara: ¿8c6976e5... == BD.hash? → ✓ SÍ
```

### Registro
```
"MiPassword123"  →  SHA-256  →  a7f3c2b8...
                                    ↓
                            Guardar en BD
                            (nunca la password)
```

### Seguridad
```
❌ sin hash:          ✅ con SHA-256:
password visible      hash visible
hackeo = acceso       hackeo = nada
```

---

## 📚 Archivos Clave

- **index.html** → Contiene la función sha256()
- **users.json** → Base de datos con hashes
- **credenciales.txt** → Contraseñas originales (referencia)

---

## 🚀 Mejoras Futuras

1. **SALT** → Agregar valor aleatorio por usuario
2. **Rate Limiting** → Limitar intentos de login
3. **HTTPS** → Encriptación en tránsito
4. **bcrypt/argon2** → Funciones más lentas (mejor seguridad)
5. **2FA** → Autenticación de dos factores

---

## 🎯 Punto Principal de la Exposición

> "El hasheo SHA-256 permite almacenar contraseñas de forma segura: 
> la contraseña original NUNCA se guarda, 
> solo un número imposible de reversar. 
> Si alguien hackea la BD, solo ve números inútiles."

