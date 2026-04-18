# 🚀 Guía de Despliegue

## 🐳 Imágenes Docker - Opciones de Despliegue

Tienes varias opciones para manejar las imágenes Docker:

### Opción 1: Construir directamente en el servidor (Recomendado - Más simple)

**Ventajas:**
- No necesitas registry
- Siempre construyes con el código más reciente
- Más control sobre el proceso

**Proceso:**
1. Sube el código al servidor (Git, scp, rsync, etc.)
2. Construye la imagen directamente en el servidor
3. Ejecuta el contenedor

```bash
# En el servidor
cd /ruta/al/proyecto
docker-compose -f docker-compose.prod.yml build
docker-compose -f docker-compose.prod.yml up -d
```

### Opción 2: Usar Docker Registry (Docker Hub, GitHub Container Registry, etc.)

**Ventajas:**
- Puedes versionar imágenes
- Más rápido si la imagen ya está construida
- Útil para CI/CD

**Proceso:**

#### 2.1 Construir y subir imagen

```bash
# En tu máquina local o CI/CD
docker build -t tu-usuario/grupero-landing:latest .
docker push tu-usuario/grupero-landing:latest
```

#### 2.2 Usar imagen pre-construida en docker-compose

Edita `docker-compose.prod.yml`:

```yaml
services:
  grupero-landing:
    image: tu-usuario/grupero-landing:latest  # En lugar de build
    # ... resto de configuración
```

#### 2.3 En el servidor

```bash
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d
```

### Opción 3: Guardar imagen como archivo .tar

**Para servidores sin acceso a internet o registries:**

```bash
# En tu máquina local
docker build -t grupero-landing:latest .
docker save grupero-landing:latest -o grupero-landing.tar

# Subir al servidor
scp grupero-landing.tar usuario@servidor:/ruta/

# En el servidor
docker load -i grupero-landing.tar
docker-compose -f docker-compose.prod.yml up -d
```

## 🔄 Proceso de Despliegue Completo

### Método Recomendado: Git + Build en Servidor

#### 1. En tu máquina local:

```bash
# Hacer commit y push
git add .
git commit -m "Initial commit"
git remote add origin <tu-repositorio-git>
git push -u origin main
```

#### 2. En tu servidor:

```bash
# Clonar el repositorio
cd /ruta/donde/quieres/el/proyecto
git clone <tu-repositorio-git> grupero-landing
cd grupero-landing

# Crear archivo .env con tu dominio
cp .env.example .env
nano .env  # Edita DOMAIN=tu-dominio.com

# Asegurar que la red web existe
docker network create web 2>/dev/null || true

# Construir la imagen Docker y desplegar
# La imagen se construye localmente en el servidor
docker-compose -f docker-compose.prod.yml up -d --build
```

**Nota:** La imagen Docker se construye directamente en el servidor. No necesitas subirla por separado.

#### 3. Para actualizar después de cambios:

```bash
# En el servidor
cd /ruta/al/proyecto
git pull

# Reconstruir la imagen con los nuevos cambios
docker-compose -f docker-compose.prod.yml up -d --build

# O si quieres forzar reconstrucción sin cache
docker-compose -f docker-compose.prod.yml build --no-cache
docker-compose -f docker-compose.prod.yml up -d
```

**¿Dónde se guardan las imágenes Docker?**
- Las imágenes se guardan en el sistema de archivos de Docker del servidor
- Por defecto en: `/var/lib/docker/` (no necesitas gestionarlas manualmente)
- Puedes verlas con: `docker images`
- Puedes eliminarlas con: `docker rmi <image-id>` si necesitas espacio

### Método Alternativo: Subir archivos directamente (Sin Git)

```bash
# En tu máquina local, comprimir el proyecto
tar -czf grupero-landing.tar.gz \
  --exclude='node_modules' \
  --exclude='dist' \
  --exclude='.astro' \
  --exclude='.env' \
  --exclude='.git' \
  .

# Subir al servidor (usando scp, rsync, etc.)
scp grupero-landing.tar.gz usuario@servidor:/ruta/destino/

# En el servidor
cd /ruta/destino
tar -xzf grupero-landing.tar.gz
cd grupero-landing
cp .env.example .env
nano .env  # Configura tu dominio

# La imagen Docker se construye aquí en el servidor
docker-compose -f docker-compose.prod.yml up -d --build
```

**Nota importante:** Con este método, la imagen Docker se construye en el servidor. No subes la imagen pre-construida, solo el código fuente.

## 📋 Checklist de Despliegue

- [ ] Repositorio Git configurado (o archivos listos para subir)
- [ ] Archivo `.env` creado con `DOMAIN=tu-dominio.com`
- [ ] Red Docker `web` existe en el servidor
- [ ] Traefik está corriendo y configurado
- [ ] DNS apunta al servidor
- [ ] Docker y Docker Compose instalados en el servidor
- [ ] Código fuente subido al servidor (Git o archivos)
- [ ] Listo para construir la imagen Docker en el servidor

## 💾 Gestión de Imágenes Docker

### Ver imágenes construidas

```bash
docker images | grep grupero-landing
```

### Limpiar imágenes antiguas (liberar espacio)

```bash
# Ver imágenes no usadas
docker images --filter "dangling=true"

# Eliminar imágenes no usadas
docker image prune

# Eliminar una imagen específica
docker rmi <image-id>
```

### Ver tamaño de imágenes

```bash
docker system df
```

## 🔍 Verificar Despliegue

```bash
# Ver estado de contenedores
docker-compose -f docker-compose.prod.yml ps

# Ver logs
docker-compose -f docker-compose.prod.yml logs -f

# Verificar que Traefik detectó el servicio
# (revisa el dashboard de Traefik o logs de Traefik)
```

## 🛠️ Comandos Útiles

```bash
# Detener
docker-compose -f docker-compose.prod.yml down

# Reiniciar
docker-compose -f docker-compose.prod.yml restart

# Reconstruir después de cambios
docker-compose -f docker-compose.prod.yml up -d --build

# Ver logs en tiempo real
docker-compose -f docker-compose.prod.yml logs -f grupero-landing
```
