# Deployment Guide - Grupero Landing

## Stack

- **Framework:** Astro 5.x
- **Node.js:** 22+ (requerido por Astro)
- **Docker:** Multi-stage build
- **Proxy:** Traefik
- **Server:** Saiynas

## Desarrollo Local

```bash
# Con Docker
docker-compose -f docker-compose.dev.yml up -d

# Sin Docker
npm install
npm run dev
```

Acceder en: `http://localhost:4321`

## Producción

### Primer Deploy

```bash
# En el servidor
cd /home/modavidc
git clone https://github.com/modavidc/grupero_landing.git
cd grupero_landing

# Configurar dominio
cp .env.example .env
# Editar DOMAIN si es necesario

# Crear red (si no existe)
docker network create web 2>/dev/null || true

# Desplegar
docker-compose -f docker-compose.prod.yml up -d --build
```

### Actualizar

```bash
cd /path/to/grupero_landing
git pull
docker-compose -f docker-compose.prod.yml up -d --build
```

### Forzar reconstrucción completa

```bash
docker-compose -f docker-compose.prod.yml down
docker-compose -f docker-compose.prod.yml up -d --build --force-recreate
```

### Reconstruir sin caché (para cambios de Node.js)

```bash
docker-compose -f docker-compose.prod.yml build --no-cache
docker-compose -f docker-compose.prod.yml up -d
```

## Verificar Deployment

```bash
# Estado del contenedor
docker-compose -f docker-compose.prod.yml ps

# Logs
docker-compose -f docker-compose.prod.yml logs -f

# Ver imagen creada
docker images | grep grupero
```

## URLs

- **Producción:** https://grupero.modavidc.com
- **Repositorio:** https://github.com/modavidc/grupero_landing

## Traefik Labels

El `docker-compose.prod.yml` configura:

- Router HTTPS en entrypoint `websecure`
- Certificado SSL con Let's Encrypt (`http-resolv`)
- Router HTTP con middleware de redirección (requiere config global de Traefik)

## Pendientes

- [ ] Configurar redirección HTTP→HTTPS global en Traefik
