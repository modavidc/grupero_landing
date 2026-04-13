# Learnings: Setup Inicial Grupero Landing

**Fecha:** 2026-04-13

## Resumen

Setup completo de landing page para Grupero con Astro, Docker y Traefik, basado en el proyecto `el_estoico_landing`.

## Lo que se hizo

### 1. Setup Inicial
- Creación de repositorio en GitHub
- Configuración de git y primer push
- Archivos base: README.md, QUICK_START.md, DEPLOY.md, .env.example

### 2. Docker
- `Dockerfile` para producción (multi-stage build)
- `Dockerfile.dev` para desarrollo con hot reload
- `docker-compose.prod.yml` con labels de Traefik
- `docker-compose.dev.yml` para desarrollo local

### 3. SEO Implementado
- Meta tags: title, description, canonical
- Open Graph (Facebook)
- Twitter Cards
- Schema.org JSON-LD (SoftwareApplication)
- Sitemap con `@astrojs/sitemap`
- robots.txt

### 4. Optimizaciones
- **Fuentes:** Preload y carga asíncrona de Google Fonts
- **Share buttons:** WhatsApp, X, Facebook, LinkedIn, Copy link
- **Tildes:** Corrección en todas las páginas (FAQ, contacto, ayuda)

### 5. Traefik
- Labels para HTTPS con Let's Encrypt
- Intento de redirección HTTP→HTTPS por servicio

## Problemas Encontrados y Soluciones

### Node.js version mismatch
**Problema:** Astro 5.x requiere Node 22+, pero Docker usaba imagen cacheada con Node 20.

**Solución:**
```bash
docker-compose -f docker-compose.dev.yml build --no-cache
```

### Redirección HTTP a HTTPS no funciona
**Problema:** Los labels de middleware en docker-compose no aplican la redirección.

**Causa probable:** El entrypoint `web` de Traefik no está configurado o no tiene el redirect global.

**Solución recomendada:** Configurar en `traefik.yml`:
```yaml
entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
          permanent: true
```

### Push a GitHub falla
**Problema:** No se puede hacer push desde Claude Code por autenticación.

**Solución:** Ejecutar `git push` manualmente en terminal.

## Comandos Útiles

```bash
# Desarrollo
docker-compose -f docker-compose.dev.yml up -d
docker-compose -f docker-compose.dev.yml logs -f

# Producción
docker-compose -f docker-compose.prod.yml up -d --build

# Reconstruir sin caché
docker-compose -f docker-compose.prod.yml build --no-cache

# Ver estado
docker ps --filter "name=grupero"
```

## Estructura Final

```
grupero_landing/
├── src/
│   ├── components/
│   │   ├── ShareButtons.astro  # Nuevo
│   │   └── ...
│   ├── layouts/
│   │   └── Layout.astro        # SEO + fonts optimizados
│   └── pages/
├── docs/
│   ├── DEPLOYMENT.md
│   └── learnings/
├── docker-compose.dev.yml
├── docker-compose.prod.yml
├── Dockerfile
├── Dockerfile.dev
├── .env.example
├── README.md
├── QUICK_START.md
└── DEPLOY.md
```

## Referencias

- [Traefik HTTP to HTTPS redirect](https://jensknipper.de/blog/traefik-http-to-https-redirect/)
- [Astro Documentation](https://docs.astro.build)
- [Traefik Docker Labels](https://doc.traefik.io/traefik/reference/routing-configuration/other-providers/docker/)
