# DOCUMENTACIÓN TÉCNICA COMPLETA DEL REPOSITORIO NESGESFinance.mempool.space

## 1. RESUMEN EJECUTIVO

**Nombre del Proyecto:** NESGESFinance.mempool.space  
**Versión:** 3.4-dev  
**Tipo:** Visualizador de Mempool de Bitcoin y Explorador de Blockchain  
**Licencia:** GNU Affero General Public License v3.0  
**Base:** Fork del proyecto open-source Mempool (mempool.space)  
**Propósito:** Plataforma institucional para visualización y análisis de la mempool de Bitcoin, exploración de blockchain, estadísticas de minería, y soporte para Lightning Network y protocolos de activos Bitcoin (Ordinals, Runes, Taproot Assets).

## 2. ARQUITECTURA GENERAL DEL SISTEMA

### 2.1 Stack Tecnológico Principal

- **Backend:** Node.js 16+ con TypeScript 4.9.3
- **Frontend:** Angular 20.3.25 con TypeScript
- **Módulo de Alto Rendimiento:** Rust (algoritmo getBlockTemplate)
- **Base de Datos:** MariaDB 10.5.21
- **Caché:** Redis + Disk Cache + Memory Cache
- **Despliegue:** Docker Compose (3 servicios: web, api, db)
- **CI/CD:** GitHub Actions con pipeline institucional personalizado

### 2.2 Arquitectura de Tres Capas

1. **Capa de Presentación (Frontend):**
   - Angular SPA con Server-Side Rendering (SSR)
   - Múltiples archivos TypeScript con componentes modulares
   - Componentes Angular para visualización interactiva
   - Soporte multi-idioma (i18n)

2. **Capa de Lógica de Negocio (Backend):**
   - API RESTful con Express.js
   - WebSocket para actualizaciones en tiempo real
   - Arquitectura modular en TypeScript
   - Modo cluster para múltiples procesos

3. **Capa de Datos:**
   - MariaDB para almacenamiento persistente
   - Redis para caché de alto rendimiento
   - Módulo Rust para operaciones críticas

## 3. BACKEND - ESTRUCTURA Y COMPONENTES

### 3.1 Estructura de Directorios

```
/backend/
├── src/
│   ├── index.ts                 # Punto de entrada principal
│   ├── config.ts                # Configuración del sistema
│   ├── api/                     # Controladores API REST
│   │   ├── bitcoin/             # APIs de Bitcoin Core
│   │   ├── lightning/           # APIs de Lightning Network
│   │   ├── liquid/              # APIs de Liquid Sidechain
│   │   ├── mining/              # Estadísticas de minería
│   │   ├── explorer/            # Explorador de blockchain
│   │   ├── acceleration/        # Servicio de aceleración de transacciones
│   │   └── websocket-handler.ts # Manejador WebSocket
│   ├── repositories/            # Capa de acceso a datos
│   ├── tasks/                   # Tareas programadas y background
│   └── cluster-mempool/         # Clustering de transacciones
├── package.json                 # Dependencias y scripts
└── Dockerfile                   # Imagen Docker para backend
```

### 3.2 Dependencias Principales del Backend

**Frameworks y Servidores:**
- `express` (4.22+): Framework HTTP
- `ws` (8.21+): WebSocket server
- `compression`: Compresión de respuestas HTTP

**Base de Datos y Caché:**
- `mysql2` (3.20+): Cliente MySQL/MariaDB
- `socks-proxy-agent`: Proxy SOCKS para Tor

**Bitcoin y Blockchain:**
- `bitcoinjs-lib`: Biblioteca Bitcoin JavaScript
- `@mempool/electrum-client`: Cliente Electrum
- `rust-gbt`: Módulo Rust para getBlockTemplate

**Utilidades:**
- `express-rate-limit`: Limitación de tasa
- `maxmind`: Geolocalización IP
- `axios` (1.16+): Cliente HTTP

### 3.3 Archivos Clave del Backend

#### 3.3.1 `/backend/src/index.ts`
Punto de entrada principal que:
- Inicializa el modo cluster para múltiples procesos worker
- Configura el servidor Express
- Monta todos los routers de API
- Inicia el servidor WebSocket
- Maneja señales de apagado graceful

#### 3.3.2 `/backend/src/config.ts`
Interface de configuración que define:
- Configuración de redes (mainnet, testnet, signet, liquid)
- Backends soportados (esplora, electrum, none)
- Configuración de Lightning Network
- Configuración de RPC de Bitcoin Core
- Parámetros de base de datos y caché
- Configuración de minería y pools

#### 3.3.3 `/backend/src/api/`
Contiene todos los controladores de API:
- **bitcoin/**: Bloques, transacciones, mempool, fees
- **lightning/**: Nodos, canales, rutas Lightning
- **liquid/**: Assets, pegs Liquid
- **mining/**: Hashrate, dificultad, pools, recompensas
- **explorer/**: Búsqueda, direcciones, UTXOs
- **acceleration/**: Servicio de aceleración de transacciones

### 3.4 Scripts del Backend

```json
"scripts": {
  "tsc": "tsc",
  "build": "tsc",
  "serve": "node --max-old-space-size=4096 dist/index.js",
  "start": "node dist/index.js",
  "start-production": "node --max-old-space-size=16384 dist/index.js",
  "test": "..."
}
```

## 4. FRONTEND - ESTRUCTURA Y COMPONENTES

### 4.1 Estructura de Directorios

```
/frontend/
├── src/
│   ├── app/
│   │   ├── components/          # 137+ componentes Angular
│   │   │   ├── mempool-block/   # Visualización de bloques
│   │   │   ├── master-page/     # Layout principal
│   │   │   ├── mining-dashboard/# Dashboard de minería
│   │   │   ├── lightning/       # Lightning Network UI
│   │   │   └── ...
│   │   ├── services/            # Servicios Angular
│   │   │   ├── api.service.ts   # Cliente API
│   │   │   ├── state.service.ts # Gestión de estado
│   │   │   └── websocket.service.ts # WebSocket client
│   │   ├── modules/             # Módulos Angular
│   │   ├── interfaces/          # Interfaces TypeScript
│   │   └── app.module.ts        # Módulo raíz
│   ├── assets/                  # Recursos estáticos
│   ├── resources/               # i18n y configuración
│   └── index.html               # HTML principal
├── package.json                 # Dependencias frontend
├── angular.json                 # Configuración Angular
└── Dockerfile                   # Imagen Docker frontend
```

### 4.2 Dependencias Principales del Frontend

**Framework Angular:**
- `@angular/core` (20.3.18): Framework principal
- `@angular/router` (20.3.18): Enrutamiento
- `@angular/platform-browser` (20.3.18): Soporte navegador
- `@angular/ssr` (20.3.18): Server-Side Rendering

**UI y Visualización:**
- `bootstrap` (5.3.8): Framework CSS
- `echarts` (5.6.0): Gráficos y visualizaciones
- `ngx-echarts` (20.0.0): Integración ECharts con Angular
- `qrcode` (1.5.5): Generación de códigos QR

**Utilidades:**
- `rxjs` (7.8.1): Programación reactiva
- `clipboard` (2.0.12): Copiar al portapapeles
- `domino` (2.1.8): Emulación DOM para SSR

### 4.3 Componentes Principales

**Visualización de Mempool:**
- `mempool-block-component`: Bloques de mempool en tiempo real
- `mempool-graph-component`: Gráfico de tamaño de mempool
- `fee-distribution-graph-component`: Distribución de fees

**Explorador de Blockchain:**
- `block-component`: Detalles de bloque
- `transaction-component`: Detalles de transacción
- `address-component`: Información de dirección

**Mining Dashboard:**
- `mining-dashboard-component`: Dashboard principal de minería
- `hashrate-chart-component`: Gráfico de hashrate
- `difficulty-adjustments-table-component`: Ajustes de dificultad
- `pool-ranking-component`: Ranking de pools

**Lightning Network:**
- `lightning-dashboard-component`: Dashboard de Lightning
- `node-component`: Detalles de nodo
- `channel-component`: Información de canal

### 4.4 Scripts del Frontend

```json
"scripts": {
  "ng": "ng",
  "serve": "ng serve",
  "serve:local-prod": "ng serve -c local-prod",
  "build": "ng build",
  "build:ssr": "ng build && ng run mempool-frontend:server",
  "serve:ssr": "node dist/mempool-frontend/server/server.mjs",
  "test": "ng test",
  "cypress:open": "cypress open",
  "cypress:run": "cypress run"
}
```

## 5. MÓDULO RUST - ALTO RENDIMIENTO

### 5.1 Ubicación y Propósito

**Ruta:** `/rust/gbt/`  
**Propósito:** Implementación de alto rendimiento del algoritmo getBlockTemplate (GBT) para construcción de bloques Bitcoin.

### 5.2 Arquitectura del Módulo Rust

**Cargo.toml:**
```toml
[package]
name = "rust-gbt"
version = "0.5.2"
edition = "2021"

[lib]
crate-type = ["cdylib"]

[dependencies]
napi = { version = "2.16.15", features = ["napi9"] }
napi-derive = "2.16.15"
```

**Características:**
- Compilado como biblioteca dinámica C (`cdylib`)
- Bindings Node.js usando `napi` (Node-API)
- Optimizaciones de release con LTO (Link-Time Optimization)
- Zero-copy marshalling entre Rust y JavaScript

### 5.3 Integración con Backend

El módulo se importa en el backend TypeScript:
```typescript
import { gbt } from 'rust-gbt';
```

Usado para operaciones críticas de rendimiento en la construcción de templates de bloques.

## 6. BASE DE DATOS Y MODELOS DE DATOS

### 6.1 Tecnología de Base de Datos

**Motor:** MariaDB 10.5.21  
**Configuración Docker:**
- Imagen: `mariadb:10.5.21`
- Puerto: 3306
- Volúmenes persistentes para `/var/lib/mysql`
- Health checks configurados

### 6.2 Esquemas Principales

La base de datos almacena:
- **Bloques:** Hash, altura, timestamp, transacciones, minero
- **Transacciones:** Txid, inputs, outputs, fees, peso
- **Pools de Minería:** Identificación, bloques minados, hashrate estimado
- **Lightning Network:** Nodos, canales, capacidad
- **Liquid:** Assets, pegs, transacciones confidenciales
- **Clustering:** Clusters de transacciones relacionadas
- **Accelerations:** Solicitudes de aceleración de transacciones

## 7. DOCKER Y DESPLIEGUE

### 7.1 Docker Compose

**Archivo:** `/docker/docker-compose.yml`

**Servicios:**

1. **web (Frontend):**
   - Build: `./frontend`
   - Puerto: 80
   - Restart: unless-stopped
   - Health check: HTTP GET /

2. **api (Backend):**
   - Build: `./backend`
   - Puerto: 8999
   - Restart: unless-stopped
   - Health check: HTTP GET /api/v1/health
   - Depends on: db

3. **db (MariaDB):**
   - Image: mariadb:10.5.21
   - Puerto: 3306
   - Restart: unless-stopped
   - Volumes: persistentes para datos
   - Health check: mysqladmin ping

### 7.2 Despliegue en Producción

**Directorio:** `/production/`

Contiene scripts y configuraciones para:
- Inicialización de base de datos
- Configuración de Nginx como reverse proxy
- Scripts de backup y restore
- Configuración de systemd services
- Gestión de certificados SSL/TLS

## 8. CI/CD Y PIPELINE INSTITUCIONAL

### 8.1 GitHub Actions

**Archivo:** `/.github/workflows/NesgesFinance_pipeline_bitcoin.yaml`

**Características del Pipeline:**
- Validación de modelos de datos JSON en `/model/`
- Variables de entorno para Bitcoin mainnet
- Configuración de Ordinals, Runes, Lightning
- Branding y configuración institucional NESGESFinance
- Despliegue automatizado

### 8.2 Otros Workflows

El repositorio incluye workflows adicionales para:
- Build y test del backend
- Build y test del frontend
- Compilación del módulo Rust
- E2E testing con Cypress
- Linting y formateo de código

## 9. REDES Y PROTOCOLOS SOPORTADOS

### 9.1 Redes Bitcoin

- **Mainnet:** Red principal de Bitcoin
- **Testnet:** Red de pruebas legacy
- **Testnet4:** Nueva red de pruebas
- **Signet:** Red de pruebas con firmas

### 9.2 Protocolos y Capas Adicionales

- **Lightning Network:** Red de pagos de segunda capa
- **Liquid Network:** Sidechain federado
- **Ordinals:** Inscripciones en satoshis
- **Runes:** Tokens fungibles en Bitcoin
- **Taproot Assets:** Activos en Taproot

## 10. APIs Y ENDPOINTS

### 10.1 API REST

**Base URL:** `/api/v1/`

**Categorías de Endpoints:**

**Bitcoin Core:**
- `GET /blocks` - Lista de bloques recientes
- `GET /block/:hash` - Detalles de bloque
- `GET /tx/:txid` - Detalles de transacción
- `GET /address/:address` - Información de dirección
- `GET /fees/recommended` - Fees recomendados

**Mining:**
- `GET /mining/pools` - Pools de minería
- `GET /mining/hashrate` - Hashrate de la red
- `GET /mining/difficulty-adjustments` - Ajustes de dificultad

**Lightning:**
- `GET /lightning/nodes` - Nodos Lightning
- `GET /lightning/channels` - Canales Lightning
- `GET /lightning/network` - Estado de la red

**Liquid:**
- `GET /liquid/assets` - Assets de Liquid
- `GET /liquid/pegs` - Pegs de Liquid

### 10.2 WebSocket

**Endpoint:** `wss://domain/api/v1/ws`

**Mensajes en Tiempo Real:**
- Nuevas transacciones en mempool
- Bloques confirmados
- Actualizaciones de fees
- Cambios en Lightning Network
- Aceleración de transacciones

## 11. LENGUAJES Y FORMATOS DE ARCHIVO

### 11.1 Lenguajes de Programación

**TypeScript (.ts):**
- Backend: 131 archivos
- Frontend: 737 archivos
- Total: ~868 archivos TypeScript
- Lógica de negocio, servicios, componentes, interfaces

**JavaScript (.js):**
- Scripts de configuración
- Tests E2E con Cypress
- Build scripts

**Rust (.rs):**
- Módulo `gbt` de alto rendimiento
- ~10 archivos en `/rust/gbt/src/`
- Algoritmos críticos de rendimiento

**HTML (.html):**
- Templates de componentes Angular
- Index.html principal
- Email templates

**CSS/SCSS (.css, .scss):**
- Estilos de componentes
- Estilos globales
- Temas y variables CSS

### 11.2 Formatos de Configuración

**JSON (.json):**
- `package.json`: Dependencias y scripts
- `tsconfig.json`: Configuración TypeScript
- `angular.json`: Configuración Angular
- Archivos de i18n (internacionalización)

**YAML (.yml, .yaml):**
- `docker-compose.yml`: Orquestación Docker
- GitHub Actions workflows
- Configuración de CI/CD

**TOML (.toml):**
- `Cargo.toml`: Configuración del proyecto Rust
- Dependencias Rust

**Markdown (.md):**
- `README.md`: Documentación principal
- Guías de instalación y uso
- Documentación de API

**Dockerfile:**
- Definición de imágenes Docker
- Frontend, backend, y servicios adicionales

## 12. CARACTERÍSTICAS INSTITUCIONALES NESGESFINANCE

### 12.1 Branding Personalizado

- Logo y colores corporativos
- Configuración de marca en frontend
- Variables de entorno institucionales

### 12.2 Pipeline Bitcoin Institucional

**Archivo:** `NesgesFinance_pipeline_bitcoin.yaml`

Validaciones específicas para:
- Modelos de datos canónicos de Bitcoin
- Integridad de datos institucionales
- Compliance y auditoría
- Despliegue institucional controlado

### 12.3 Soporte para Activos Bitcoin

- **Ordinals:** Tracking e indexación de inscripciones
- **Runes:** Soporte para tokens Runes
- **Taproot Assets:** Integración de activos Taproot

## 13. DESARROLLO Y TESTING

### 13.1 Entorno de Desarrollo

**Requisitos:**
- Node.js 16+
- Rust 1.70+
- MariaDB 10.5+
- Redis
- Docker y Docker Compose

**Setup Local:**
```bash
# Backend
cd backend
npm install
npm run build
npm run serve

# Frontend
cd frontend
npm install
npm run serve

# Rust module
cd rust/gbt
cargo build --release
```

### 13.2 Testing

**Backend:**
- Unit tests con Jest
- Integration tests

**Frontend:**
- Unit tests con Karma/Jasmine
- E2E tests con Cypress
- Component tests

**Cypress E2E:**
- Tests de flujos completos
- Tests de visualización
- Tests de API

## 14. CACHÉ Y OPTIMIZACIÓN

### 14.1 Estrategias de Caché

**Redis Cache:**
- Mempool data
- Block summaries
- Fee estimates
- Lightning network state

**Disk Cache:**
- Historical blocks
- Transaction data
- Mining statistics

**Memory Cache:**
- Hot data en proceso
- Estado de aplicación
- WebSocket connections

### 14.2 Optimizaciones de Rendimiento

- **Cluster Mode:** Múltiples procesos worker
- **Rust Module:** Operaciones críticas en Rust
- **HTTP Compression:** Compresión gzip/brotli
- **CDN:** Assets estáticos servidos vía CDN
- **Lazy Loading:** Carga diferida de módulos Angular
- **Rate Limiting:** Protección contra abuso de API

## 15. SEGURIDAD

### 15.1 Medidas de Seguridad

- **HTTPS:** Comunicación encriptada (en producción)
- **CORS:** Control de acceso entre orígenes
- **Rate Limiting:** Limitación de peticiones por IP
- **Input Validation:** Validación de todos los inputs
- **SQL Injection Protection:** Prepared statements
- **XSS Protection:** Sanitización de HTML
- **CSRF Protection:** Tokens CSRF

### 15.2 Privacidad

- No tracking de usuarios
- Sin cookies de terceros
- Opcional: conexión via Tor
- Logs mínimos en producción

## 16. MONITORIZACIÓN Y LOGS

### 16.1 Logging

**Backend:**
- Logs estructurados
- Niveles: error, warn, info, debug
- Rotación de logs

**Frontend:**
- Error tracking
- Performance monitoring

### 16.2 Health Checks

- API health endpoint: `/api/v1/health`
- Database connectivity check
- Redis connectivity check
- Bitcoin node connectivity check

## 17. ESCALABILIDAD

### 17.1 Escalado Horizontal

- Múltiples instancias del backend
- Load balancer (Nginx)
- Session-less design

### 17.2 Escalado Vertical

- Configuración de memoria: `--max-old-space-size=16384`
- Cluster mode para uso de todos los CPUs
- Database connection pooling

## 18. LICENCIA Y CONTRIBUCIÓN

### 18.1 Licencia

**GNU Affero General Public License v3.0 (AGPL-3.0)**

Requiere:
- Divulgación del código fuente
- Licencia de obras derivadas bajo AGPL-3.0
- Notificación de cambios
- Divulgación de código fuente para uso en red

### 18.2 Contribución

Basado en el proyecto open-source Mempool con:
- Modificaciones institucionales NESGESFinance
- Features personalizados
- Branding corporativo

## 19. CONCLUSIÓN

El repositorio NESGESFinance.mempool.space es una plataforma completa y profesional para:

✅ **Visualización de Mempool Bitcoin** en tiempo real  
✅ **Exploración de Blockchain** completa  
✅ **Estadísticas de Minería** detalladas  
✅ **Lightning Network Explorer**  
✅ **Soporte Multi-Red** (mainnet, testnet, signet, liquid)  
✅ **Protocolos Bitcoin Modernos** (Ordinals, Runes, Taproot Assets)  
✅ **Arquitectura Escalable** (Docker, Cluster, Caché)  
✅ **Stack Tecnológico Moderno** (TypeScript, Angular, Rust)  
✅ **Pipeline CI/CD Institucional**  
✅ **Alta Performance** (Módulo Rust para operaciones críticas)

La arquitectura multi-capa, el uso de tecnologías modernas, y la integración de un módulo Rust para alto rendimiento hacen de esta plataforma una solución robusta para instituciones financieras que requieren análisis profesional de Bitcoin.

---

**Fecha de Documentación:** 16 de julio de 2026  
**Versión del Repositorio:** 3.4-dev  
**Branch:** copilot  
**Autor:** NESGESFinance Team
