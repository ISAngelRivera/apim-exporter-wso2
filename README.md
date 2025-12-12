# apim-exporter-wso2

Procesador específico para WSO2 API Manager en la arquitectura APIOps GitOps.

## Arquitectura

```
┌─────────────────────┐
│  Publisher Portal   │
│  (UATRegistration)  │
└─────────┬───────────┘
          │ workflow_dispatch
          ▼
┌─────────────────────┐
│   apim-exporter-wso2    │  ← Este repositorio
│  (GitHub Actions)   │
│  - Exporta API      │
│  - Crea PR          │
└─────────┬───────────┘
          │ Pull Request con datos PLANOS
          ▼
┌─────────────────────┐
│ apim-apiops-controller │
│  - Linters          │
│  - CRQ Helix        │
│  - Almacena API     │
└─────────────────────┘
```

## Workflows

### `receive-uat-request.yml`

Recibe solicitudes de registro UAT desde el botón en WSO2 Publisher.

**Trigger**: `workflow_dispatch`

**Inputs**:
| Input | Requerido | Descripción |
|-------|-----------|-------------|
| `apiId` | Sí | ID de la API en WSO2 |
| `apiName` | Sí | Nombre de la API |
| `apiVersion` | Sí | Versión de la API |
| `apiContext` | No | Contexto (path) de la API |
| `userId` | No | Usuario que dispara el registro |
| `metadata` | No | Metadatos adicionales (JSON) |

**Output**: PR en apim-apiops-controller con estructura de datos PLANOS:
```
requests/
└── 2024-12-04-pizzaapi-v1-0-0-uat/
    ├── request.yaml      # Metadatos de la solicitud
    ├── api.yaml          # Definición de la API (exportada de WSO2)
    ├── swagger.yaml      # OpenAPI spec
    └── params.yaml       # Configuración de entorno
```

## Configuración

### Secrets requeridos

| Secret | Descripción |
|--------|-------------|
| `WSO2_BASE_URL` | URL base de WSO2 APIM (default: https://localhost:9443) |
| `WSO2_USERNAME` | Usuario admin de WSO2 |
| `WSO2_PASSWORD` | Password de WSO2 |
| `GIT_HELIX_PAT` | Personal Access Token para crear PRs en apim-apiops-controller |
| `GIT_HELIX_REPO` | Repositorio destino (default: ISAngelRivera/apim-apiops-controller) |

### Token del Publisher

El botón UATRegistration en el Publisher UI necesita un token de GitHub para disparar workflows.

Configurar en localStorage del navegador:
```javascript
localStorage.setItem('github_pat_token', 'ghp_xxx...');
```

El token necesita scope `repo` para disparar workflows.

## Formato de Request (Datos Planos)

### request.yaml
```yaml
action: register-uat
request_id: 2024-12-04-pizzaapi-v1-0-0-uat
timestamp: 2024-12-04T10:30:00Z

source:
  system: wso2
  processor: apim-exporter-wso2
  run_id: "12345"

api:
  id: "abc-123"
  name: PizzaAPI
  version: 1.0.0
  context: /pizza

user:
  id: usuario@empresa.com
```

### api.yaml
```yaml
type: api
version: v4
data:
  name: PizzaAPI
  version: 1.0.0
  context: /pizza
  provider: admin
  lifeCycleStatus: PUBLISHED
  # ... resto de la definición exportada de WSO2
```

## Notas de Diseño

Este repo es **específico de WSO2**. Para otros API Managers:

- `Apigee-Processor` - Para Google Apigee
- `Kong-Processor` - Para Kong Gateway
- `AWS-API-GW-Processor` - Para AWS API Gateway

Todos los processors crean PRs con el mismo formato en apim-apiops-controller, haciendo la arquitectura **vendor-agnostic**.
