# Git Usage - Sync Server Platform

## 📋 **Estructura del Proyecto**

Este proyecto utiliza **Git Submódulos** para gestionar múltiples repositorios:

```
sync-server-platform/ (Proyecto Principal)
├── LuckPerms/        (Submódulo)
├── rest-api/         (Submódulo)  
├── bot-discord/      (Submódulo - Nuestro desarrollo principal)
└── .gitmodules       (Configuración submódulos)
```

---

## 🚀 **Configuración Inicial**

### **Al clonar el proyecto por primera vez:**

```bash
# Clonar proyecto principal con todos los submódulos
git clone --recursive git@github.com:matienzo01/sync-server-platform.git

# O si ya lo clonaste sin --recursive:
git clone git@github.com:matienzo01/sync-server-platform.git
cd sync-server-platform
git submodule update --init --recursive
```

---

## 💻 **Flujo de Desarrollo Normal**

### **1. Trabajar en bot-discord (caso más común):**

```bash
# Ir al submódulo
cd bot-discord/

# Asegurarse de estar en master y actualizado
git checkout master
git pull

# Crear rama para nueva funcionalidad
git checkout -b feature/nueva-funcionalidad

# Hacer cambios, commits normales
git add .
git commit -m "Implementar nueva funcionalidad"

# Push de la rama
git push -u origin feature/nueva-funcionalidad
```

### **2. Merge a master (recomendado: usar Pull Request):**

**Opción A - Por GitHub (recomendado):**
1. Crear Pull Request en GitHub
2. Review y merge a master
3. Continuar con paso 3

**Opción B - Merge directo:**
```bash
git checkout master
git merge feature/nueva-funcionalidad
git push
git branch -d feature/nueva-funcionalidad  # Limpiar rama local
```

### **3. Actualizar el proyecto padre:**

```bash
# Volver al proyecto principal
cd ..

# Actualizar submódulo a último commit de master
git submodule update --remote --merge

# Commit y push de la actualización
git add bot-discord
git commit -m "Actualizar bot-discord con nueva funcionalidad"
git push
```

---

## 🔄 **Trabajar en Múltiples Máquinas**

### **Al cambiar de máquina o trabajar en equipo:**

```bash
# En la nueva máquina/sesión
cd sync-server-platform/

# Actualizar proyecto principal
git pull

# Actualizar TODOS los submódulos a las referencias del proyecto padre
git submodule update --recursive

# O si quieres traer los últimos cambios de las ramas master:
git submodule update --remote --merge
```

### **Verificar estado de submódulos:**

```bash
# Ver estado de todos los submódulos
git submodule status

# Resultado esperado:
# aad93a7... bot-discord (heads/master)
# 1f85407... LuckPerms (heads/master)  
# b4a49e0... rest-api (heads/main)
```

---

## ⚠️ **Problemas Comunes y Soluciones**

### **🔸 "Submódulo en HEAD desacoplada"**

```bash
cd bot-discord/
git status  # Si dice "HEAD desacoplada de..."

# Solución:
git checkout master
git pull
```

### **🔸 "Cambios no guardados en submódulo"**

```bash
# Ver qué submódulo tiene cambios
git status  # Mostrará "modificados: bot-discord (nuevos commits)"

# Actualizar proyecto padre
git add bot-discord
git commit -m "Actualizar bot-discord"
git push
```

### **🔸 "Error al hacer pull en otra máquina"**

```bash
# Si el proyecto padre cambió referencias de submódulos
git pull
git submodule update --recursive

# Si quieres forzar actualización a últimos commits:
git submodule update --remote --merge
```

---

## 🎯 **Flujos Específicos**

### **🔹 Desarrollo Solo en bot-discord:**

```bash
# 1. Trabajar en feature
cd bot-discord/
git checkout -b feature/mi-cambio
# hacer cambios...
git commit -m "Mi cambio" && git push -u origin feature/mi-cambio

# 2. Merge a master (por PR preferentemente)
git checkout master && git pull
git merge feature/mi-cambio && git push

# 3. Actualizar proyecto padre
cd ..
git submodule update --remote --merge
git add bot-discord && git commit -m "Actualizar bot-discord" && git push
```

### **🔹 Cambios en Múltiples Submódulos:**

```bash
# Cambiar LuckPerms
cd LuckPerms/
git checkout -b feature/cambio-luckperms
# hacer cambios...
git commit && git push && git checkout master && git merge feature/cambio-luckperms && git push

# Cambiar bot-discord  
cd ../bot-discord/
git checkout -b feature/cambio-bot
# hacer cambios...
git commit && git push && git checkout master && git merge feature/cambio-bot && git push

# Actualizar proyecto padre
cd ..
git submodule update --remote --merge
git add LuckPerms bot-discord
git commit -m "Actualizar LuckPerms y bot-discord"
git push
```

---

## 📚 **Comandos de Referencia Rápida**

### **Submódulos:**
```bash
# Ver estado de todos los submódulos
git submodule status

# Actualizar submódulos a referencias del proyecto padre
git submodule update --recursive

# Actualizar submódulos a últimos commits de sus ramas master
git submodule update --remote --merge

# Ejecutar comando en todos los submódulos
git submodule foreach 'git status'
git submodule foreach 'git checkout master && git pull'
```

### **Verificación:**
```bash
# Ver qué ha cambiado
git status
git submodule status

# Ver historial de cambios en submódulos
git log --oneline --graph --all

# Ver diferencias en submódulos
git diff --submodule
```

---

## 🎨 **Configuración Automática (.gitmodules)**

El proyecto está configurado para tracking automático:

```ini
[submodule "bot-discord"]
    path = bot-discord
    url = git@github.com:matienzo01/bot-discord.git
    branch = master  # ← Esto hace que siga automáticamente master
```

Esto significa que `git submodule update --remote` siempre traerá el último commit de master.

---

## 🚨 **Reglas Importantes**

### **✅ HACER:**
- Siempre crear ramas para nuevas funcionalidades
- Usar Pull Requests para merge a master
- Actualizar proyecto padre después de cambios en submódulos
- Hacer `git submodule update` al cambiar de máquina

### **❌ NO HACER:**
- Commitear directamente en master sin rama
- Olvidar actualizar el proyecto padre tras cambios en submódulos
- Trabajar en submódulos sin estar en una rama específica
- Hacer push del proyecto padre sin actualizar referencias de submódulos

---

## 📞 **En Caso de Problemas**

Si algo sale mal, estos comandos suelen resolver la mayoría de problemas:

```bash
# Reset completo (cuidado: pierde cambios no committeados)
git submodule foreach --recursive git reset --hard
git submodule update --init --recursive

# Verificar y reparar estado
git submodule status
git submodule sync --recursive
git submodule update --remote --merge
```

---

**💡 Tip:** Siempre verifica con `git submodule status` antes de hacer push del proyecto padre para asegurarte de que las referencias sean correctas.