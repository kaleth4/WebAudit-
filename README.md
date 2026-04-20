# 🔍 WebAudit — Análisis de Seguridad Web

```
██╗    ██╗███████╗██████╗  █████╗ ██╗   ██╗██████╗ ██╗████████╗
██║    ██║██╔════╝██╔══██╗██╔══██╗██║   ██║██╔══██╗██║╚══██╔══╝
██║ █╗ ██║█████╗  ██████╔╝███████║██║   ██║██║  ██║██║   ██║
██║███╗██║██╔══╝  ██╔══██╗██╔══██║██║   ██║██║  ██║██║   ██║
╚███╔███╔╝███████╗██████╔╝██║  ██║╚██████╔╝██████╔╝██║   ██║
 ╚══╝╚══╝ ╚══════╝╚═════╝ ╚═╝  ╚═╝ ╚═════╝ ╚═════╝ ╚═╝   ╚═╝
```

> Guía técnica de análisis de seguridad para aplicaciones web modernas (Vue/React/Vite).  
> Orientado a **Bug Bounty**, auditoría web y hardening defensivo.

---

## 🎯 Alcance del análisis

Este proyecto documenta el proceso de auditoría de seguridad sobre aplicaciones web compiladas con frameworks modernos (Vue 3, React, Vite), incluyendo:

- Análisis de HTML estático
- Inspección de bundles JS minificados
- Identificación de vectores de ataque
- Headers de seguridad
- Configuración de CDN (Cloudflare)

---

## 🛡️ Vectores analizados

### Frontend (HTML + JS)

| Vector | Descripción | Criticidad |
|---|---|---|
| CSP ausente | Sin `Content-Security-Policy`, el navegador no filtra fuentes de scripts | 🔴 Alta |
| `v-html` sin sanitizar | Directiva de Vue que puede inyectar HTML arbitrario (XSS) | 🔴 Alta |
| Variables de entorno expuestas | Llaves API filtradas en el bundle JS compilado | 🔴 Alta |
| Scripts externos sin SRI | Dependencias de terceros sin verificación de integridad | 🟡 Media |
| Lógica de roles en frontend | Condiciones `isAdmin` manipulables desde consola | 🟡 Media |
| Listado de directorios `/assets/` | Exposición de estructura interna del proyecto | 🟡 Media |

### Servidor / Infraestructura

| Vector | Descripción | Criticidad |
|---|---|---|
| Falta de `X-Frame-Options` | Vulnerable a Clickjacking via iframe | 🟡 Media |
| Falta de HSTS | Sin `Strict-Transport-Security` | 🟡 Media |
| Cloudflare Origin IP Bypass | Posible exposición de IP real del servidor | 🟠 Alta |

---

## 🔬 Metodología de análisis JS

Para analizar bundles minificados (ej: `index-B1veIbDR.js`):

```
1. DevTools → F12 → Sources → seleccionar archivo → { } (Pretty Print)
2. Buscar palabras clave críticas:
   - apiKey, password, config, secret
   - sk_live (Stripe), AIza (Google APIs)
   - /api/v1/, /admin/, /internal/
   - isAdmin, role: "admin"
   - innerHTML, v-html
```

---

## 🧰 Stack de herramientas (Kali Linux)

```bash
# Escaneo automático de vulnerabilidades web
owasp-zap                    # XSS, inyecciones, fallos de configuración

# Escaneo de servidor
nikto -h https://objetivo.com

# Análisis de dependencias npm
npm audit

# Búsqueda de secretos en JS
trufflehog filesystem ./     # Detecta API keys y credenciales
linkfinder -i bundle.js -o results.html  # Endpoints ocultos

# Escaneo de puertos
nmap -sV -p 80,443,8080 objetivo.com
```

---

## ✅ Checklist de hardening

```
[ ] Implementar Content-Security-Policy (CSP)
[ ] Configurar X-Frame-Options: DENY
[ ] Habilitar Strict-Transport-Security (HSTS)
[ ] Ejecutar npm audit y actualizar dependencias
[ ] Verificar que /assets/ no permite listado de directorios
[ ] Sanitizar todas las entradas de usuario antes de renderizar
[ ] Validar autorización en el servidor, nunca solo en el frontend
[ ] Revisar variables de entorno no filtradas en el bundle final
```

---

## 📦 Tecnologías detectadas

- **Framework**: Vue 3.5.13 (`@vue/shared`, `@vue/reactivity`)
- **Build tool**: Vite (hashes en nombres de archivo)
- **CDN**: Cloudflare Insights
- **Bundling**: ESM con modulepreload

---

## ⚠️ Disclaimer

> Este análisis es para **uso educativo y en entornos autorizados**.  
> Realizar pruebas de penetración sin permiso explícito es ilegal.  
> Aplica este conocimiento únicamente en programas de Bug Bounty o auditorías con scope definido.

---

## 📚 Referencias

- [OWASP Testing Guide v4](https://owasp.org/www-project-web-security-testing-guide/)
- [Content Security Policy — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [TruffleHog](https://github.com/trufflesecurity/trufflehog)
- [LinkFinder](https://github.com/GerbenJavado/LinkFinder)
