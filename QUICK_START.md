# 🚀 Inicio Rápido - Despliegue Simple

## Método más sencillo: Git + Build en servidor

### 1️⃣ En tu máquina local

```bash
# Hacer commit inicial
git add .
git commit -m "Initial commit"

# Si aún no tienes remoto
git remote add origin <tu-repositorio-git>
git push -u origin main
```

### 2️⃣ En tu servidor

```bash
# Clonar proyecto
cd /ruta/donde/quieres/el/proyecto
git clone <tu-repositorio-git> grupero-landing
cd grupero-landing

# Configurar dominio
cp .env.example .env
echo "DOMAIN=tu-dominio.com" > .env

# Crear red si no existe
docker network create web 2>/dev/null || true

# Construir y desplegar (todo en uno)
docker-compose -f docker-compose.prod.yml up -d --build
```

### 3️⃣ Para actualizar después

```bash
# En el servidor
cd /ruta/al/proyecto
git pull
docker-compose -f docker-compose.prod.yml up -d --build
```

## ✅ Listo

La imagen Docker se construye automáticamente en el servidor. No necesitas hacer nada más.

## 🔍 Verificar

```bash
# Ver que está corriendo
docker-compose -f docker-compose.prod.yml ps

# Ver logs
docker-compose -f docker-compose.prod.yml logs -f
```

## 📝 Notas

- La imagen Docker se guarda automáticamente en el servidor
- Traefik detectará el contenedor automáticamente
- El certificado SSL se generará automáticamente
- Asegúrate de que el DNS apunte al servidor
