# Inventario App - Pipeline CI/CD con Docker y Kubernetes

Aplicación web de gestión de inventario desarrollada con **Node.js** y **Express**, utilizada como base para la implementación de un pipeline completo de **Integración Continua y Despliegue Continuo (CI/CD)**.

Este proyecto forma parte de una práctica académica enfocada en la automatización del ciclo de vida del software mediante Docker, GitHub Actions, GitHub Container Registry (GHCR) y Kubernetes sobre Minikube.

## Objetivos

- Construir una imagen Docker utilizando un Dockerfile Multi-stage.
- Automatizar pruebas, construcción y publicación de imágenes mediante GitHub Actions.
- Publicar imágenes en GitHub Container Registry (GHCR).
- Desplegar la aplicación en Kubernetes utilizando Rolling Update.
- Implementar una estrategia de despliegue Blue-Green.
- Aplicar buenas prácticas de seguridad mediante Kubernetes Secrets y Trivy.

---

## Descripción general

**Inventario App** es una aplicación web desarrollada con **Node.js** y **Express** que permite gestionar un catálogo básico de productos mediante una API REST y una interfaz web estática. La información se almacena en una base de datos local en formato JSON, utilizada únicamente con fines de demostración durante la práctica.

Sobre esta aplicación se implementó un pipeline completo de **Integración Continua y Despliegue Continuo (CI/CD)**, automatizando las etapas de pruebas, construcción de imágenes Docker, publicación en GitHub Container Registry (GHCR) y despliegue en un clúster local de Kubernetes utilizando Minikube.

Como parte del laboratorio también se implementó una estrategia de despliegue **Blue-Green**, junto con buenas prácticas de infraestructura como el uso de **Kubernetes Secrets** para la gestión de credenciales y **Trivy** para el análisis automático de vulnerabilidades durante el pipeline de integración continua.

El propósito principal del proyecto es demostrar un flujo de trabajo moderno para el despliegue de aplicaciones contenerizadas, aplicando conceptos de automatización, disponibilidad, seguridad y despliegue controlado sobre una infraestructura basada en Kubernetes.
## 2. Arquitectura general

El siguiente diagrama de flujo ASCII detalla el camino que sigue el código desde el desarrollo hasta el clúster de Kubernetes, y cómo el tráfico del usuario final llega a los Pods y consume la base de datos local:

```
[ Usuario Final ]
       │ (Acceso HTTP)
       ▼
[ Service: inventario-service / inventario-blue-green ]
       │ (Redirección de tráfico)
       ▼
[ Pods (Réplicas del Contenedor) ]
       │
       ├──► [ Express App (Node.js) ] ──► [ Local JSON DB (data/products.json) ]
       │
─────────────────────────────────────────────────────────────────────────────
[ Desarrollador ]
       │ (Push a main)
       ▼
[ GitHub Repository ]
       │ (Dispara Workflow)
       ▼
[ GitHub Actions ]
       ├──► Job 1: [ build-test ]
       │           └── Ejecución de pruebas unitarias (npm test)
       │
       └──► Job 2: [ build-push ] (Requiere build-test exitoso)
                   ├── Construcción de Imagen Docker (Multi-stage)
                   ├── Escaneo de Seguridad (Trivy Scan - Severidad Crítica)
                   └── Push a Registro de Contenedores
                         │
                         ▼
             [ GitHub Container Registry (GHCR) ]
                         │
                         ▼ (Pull de Imagen)
                 [ Clúster Minikube ]
```

---

## 3. Tecnologías utilizadas

El proyecto utiliza las siguientes tecnologías y versiones específicas declaradas en la configuración y código fuente:

| Tecnología | Versión | Propósito |
| :--- | :--- | :--- |
| **Node.js** | `22` (Alpine 3.22) | Entorno de ejecución para el servidor backend, definido en la base de la imagen Docker. |
| **Express** | `^4.19.2` | Framework web y de API REST de Node.js para servir la interfaz estática y gestionar endpoints de inventario. |
| **Docker** | Multi-stage | Herramienta de contenerización de la aplicación para aislar dependencias y entornos. |
| **GitHub Actions** | N/A | Motor de automatización para ejecutar el pipeline de integración y entrega continua. |
| **Trivy** | `v0.36.0` (Action) | Analizador de vulnerabilidades de seguridad estática de la imagen Docker en el pipeline. |
| **GHCR** | N/A | Registro de contenedores (GitHub Container Registry) para alojar imágenes seguras y versionadas. |
| **Kubernetes** | API `apps/v1` / `v1` | Orquestador de contenedores para gestionar réplicas, balanceo de carga y despliegues. |
| **Minikube** | N/A | Clúster Kubernetes local de un solo nodo utilizado para la simulación y desarrollo de infraestructura. |

---

## 4. Estructura del proyecto

A continuación se muestra el árbol de directorios del proyecto con el detalle de los archivos más significativos:

```
.
├── .github/
│   └── workflows/
│       └── ci-cd.yml          # Definición del workflow de CI/CD de GitHub Actions
├── data/
│   └── products.json         # Archivo JSON que actúa como base de datos local
├── k8s/
│   ├── blue-green/
│   │   ├── blue.yaml         # Deployment Kubernetes para el entorno Blue (v1)
│   │   ├── green.yaml        # Deployment Kubernetes para el entorno Green (v2)
│   │   └── service.yaml      # Service que expone y conmuta el tráfico Blue-Green
│   ├── deployment.yaml       # Deployment Kubernetes base para Rolling Update
│   ├── secret.yaml           # Secret de Kubernetes que almacena la API_KEY
│   └── service.yaml          # Service Kubernetes base (NodePort) de acceso local
├── public/
│   ├── app.js                # Lógica del cliente para interactuar con la API REST
│   ├── index.html            # Interfaz de usuario del catálogo de productos
│   └── styles.css            # Estilos del frontend de la aplicación
├── Dockerfile                # Configuración de construcción optimizada de la imagen
├── db.js                     # Gestor de lectura y escritura del archivo JSON
├── server.js                 # Servidor backend Express y definición de endpoints
└── server.test.js            # Suite de pruebas unitarias locales del servidor
```

### Archivos clave:
* `Dockerfile`: Configura la compilación de la aplicación en dos etapas para minimizar el tamaño final.
* `server.js`: Define los endpoints del API rest y las comprobaciones de salud del servidor.
* `db.js`: Administra la lectura/escritura física en `data/products.json`, implementando el método `canAccessDb()` para comprobar la integridad.
* `server.test.js`: Contiene las pruebas de los endpoints principales (`/health`, `/version`, `/api/products`, etc.) usando el runner nativo de pruebas de Node.js.

---

## 5. Ejecución local

Para probar y validar la aplicación de forma local sin Docker o Kubernetes, siga estos pasos:

### 1. Instalar dependencias
Instale el conjunto de librerías requeridas (Express y herramientas de desarrollo):
```bash
npm install
```

### 2. Ejecutar pruebas unitarias
Ejecute la suite de pruebas unitarias nativas para verificar la integridad del código:
```bash
npm test
```

### 3. Iniciar el servidor
Levante el servidor en el puerto local por defecto (`3000`):
```bash
npm start
```

### 4. Verificación de Endpoints
Para verificar que el servicio funciona correctamente en local, puede usar herramientas como `curl`:

* **Verificar la salud del sistema (`/health`):**
  ```bash
  curl http://localhost:3000/health
  ```
  *Respuesta:* `{"status":"ok"}` (Verifica que la base de datos es accesible y no hay fallos simulados).

* **Verificar la versión activa (`/version`):**
  ```bash
  curl http://localhost:3000/version
  ```
  *Respuesta:* `{"version":"v1","color":"blue","hostname":"Nombre-Host"}` (Muestra la versión de la app, el color del banner y el hostname local).

* **Listar productos actuales (`/api/products`):**
  ```bash
  curl http://localhost:3000/api/products
  ```
  *Respuesta:* Retorna la lista en formato JSON de los productos pre-cargados (Teclado, Mouse, Monitor).

* **Agregar un nuevo producto (`POST /api/products`):**
  ```bash
  curl -X POST -H "Content-Type: application/json" -d '{"name":"Auriculares","sku":"AUR-004","stock":15,"price":29.99}' http://localhost:3000/api/products
  ```

---

## 6. Docker

El proyecto cuenta con un `Dockerfile` diseñado bajo una estrategia de **Multi-stage Build** (construcción multietapa).

### Por qué se utiliza Multi-stage Build
Esta técnica separa el entorno utilizado para compilar y probar la aplicación del entorno final donde se ejecuta. Esto permite:
1. **Reducción del tamaño de la imagen:** No se incluyen dependencias de desarrollo ni herramientas del sistema innecesarias en la imagen final.
2. **Mayor seguridad:** Reduce la superficie de ataque al no incluir código de prueba ni utilidades de compilación en el contenedor de producción.
3. **Paso de calidad garantizado:** El pipeline de construcción ejecuta las pruebas unitarias como una etapa obligatoria antes de generar el artefacto de ejecución.

### Etapas del Dockerfile

1. **Etapa 1: Builder (`builder`)**
   * Parte de la imagen base `node:22-alpine3.22`.
   * Copia los archivos de definición de dependencias (`package*.json`).
   * Ejecuta `npm ci` para instalar todas las dependencias (desarrollo y producción).
   * Copia todo el código fuente del proyecto.
   * Ejecuta `npm test` para correr los tests. Si las pruebas fallan, la construcción de la imagen se detiene de forma inmediata.

2. **Etapa 2: Producción (Runtime)**
   * Parte de una nueva instancia limpia de `node:22-alpine3.22`.
   * Configura la variable de entorno `NODE_ENV=production`.
   * Copia del `builder` únicamente los archivos indispensables para producción: `package*.json`, la carpeta `node_modules` (con dependencias de producción ya instaladas), `server.js`, `db.js`, `public/` y `data/`.
   * Expone el puerto `3000` y arranca la aplicación con `npm start`.

### Comandos de Docker

* **Construir la imagen localmente:**
  ```bash
  docker build -t ghcr.io/fernandomejia19/inventario-app:latest .
  ```

* **Ejecutar el contenedor en segundo plano:**
  ```bash
  docker run -d --name inventario-app -p 3000:3000 ghcr.io/fernandomejia19/inventario-app:latest
  ```

* **Detener y eliminar el contenedor:**
  ```bash
  docker stop inventario-app
  docker rm inventario-app
  ```

* **Verificación del contenedor:**
  ```bash
  curl http://localhost:3000/health
  ```

---

## 7. Pipeline CI/CD

El pipeline está definido en el workflow de GitHub Actions `.github/workflows/ci-cd.yml`. Se ejecuta de forma automatizada al detectar un evento `push` en la rama principal `main`.

### Diagrama del Pipeline

```
 [ Push a main ]
        │
        ▼
 ┌──────────────┐
 │  build-test  │ 
 └──────┬───────┘
        │ (Verifica código e instala dependencias)
        ▼
 ┌──────────────┐
 │   npm test   │
 └──────┬───────┘
        │ (Si las pruebas pasan correctamente)
        ▼
 ┌──────────────┐
 │  build-push  │ (Requiere build-test exitoso)
 └──────┬───────┘
        │ (Login en ghcr.io)
        ▼
 ┌──────────────┐
 │ Docker Build │ (Genera tags: SHA y latest)
 └──────┬───────┘
        │
        ▼
 ┌──────────────┐
 │  Trivy Scan  ├───────► [ Vulnerabilidades Críticas? ] ──► (Fallo / Abortar Pipeline)
 └──────┬───────┘
        │ (Escaneo exitoso)
        ▼
 ┌──────────────┐
 │  Docker Push │ (Sube imagen final a GHCR)
 └──────────────┘
```

### Detalle de los Jobs

#### Job 1: `build-test`
* Corre en `ubuntu-latest`.
* Descarga el código fuente mediante `actions/checkout@v4`.
* Configura Node.js versión `22` con `actions/setup-node@v4`.
* Instala las dependencias de forma exacta usando `npm ci`.
* Ejecuta las pruebas automatizadas con `npm test`.

#### Job 2: `build-push`
* Se ejecuta únicamente si el job `build-test` se completa con éxito (`needs: build-test`).
* Inicia sesión en el registro GHCR (`ghcr.io`) mediante `docker/login-action@v3` usando el token interno del repositorio (`secrets.GITHUB_TOKEN`).
* Construye la imagen Docker etiquetándola simultáneamente con el hash del commit (`${{ github.sha }}`) y con la etiqueta `latest`.
* **Escaneo de seguridad con Trivy:** Inspecciona la imagen compilada. Si Trivy detecta vulnerabilidades del sistema operativo de nivel `CRITICAL`, el workflow falla inmediatamente (`exit-code: "1"`) impidiendo la subida de la imagen.
* **Publicación:** Si el escaneo es limpio, se realiza el push de ambas etiquetas a GHCR.

---

## 8. Publicación en GHCR

Las imágenes construidas por el pipeline se almacenan en **GitHub Container Registry (GHCR)** bajo el namespace del usuario:
`ghcr.io/fernandomejia19/inventario-app`

### Estrategia de Etiquetado

1. **Tag por SHA del Commit:** Cada imagen se etiqueta con el hash SHA correspondiente al commit de Git que inició la ejecución (ej: `ghcr.io/fernandomejia19/inventario-app:6c507c...`). Esto asegura inmutabilidad e idoneidad para auditorías.
2. **Tag latest:** Apunta a la última versión estable que ha pasado todas las fases de pruebas y escaneo.

### Inspección de Imagen
Para descargar de forma manual la imagen publicada en GHCR, use el siguiente comando:
```bash
docker pull ghcr.io/fernandomejia19/inventario-app:latest
```

---

## 9. Kubernetes

Los manifiestos que definen el estado deseado en el clúster están ubicados en la carpeta `k8s/`.

### Manifiestos base

#### 1. `k8s/deployment.yaml`
Establece las características del despliegue de la aplicación:
* **Replicas:** Configura un mínimo de `2` Pods corriendo simultáneamente para alta disponibilidad.
* **Rolling Update Strategy:** Define cómo se actualizan las réplicas:
  * `maxUnavailable: 1`: Indica que como máximo un pod puede estar inactivo durante la actualización.
  * `maxSurge: 1`: Indica que como máximo se puede crear un pod extra por encima del número deseado durante el despliegue.
* **Probes (Sondas de salud):**
  * `readinessProbe`: Realiza peticiones HTTP GET a `/health` en el puerto 3000 cada 5 segundos (comenzando a los 5 segundos). Determina si el Pod está listo para recibir peticiones externas.
  * `livenessProbe`: Realiza peticiones HTTP GET a `/health` cada 10 segundos (comenzando a los 15 segundos). Si el endpoint falla, Kubernetes destruye el Pod y arranca uno nuevo automáticamente.

#### 2. `k8s/service.yaml`
Define un Service tipo `NodePort` llamado `inventario-service`:
* Expone el puerto `80` a nivel de clúster y lo mapea al `targetPort: 3000` de los pods que posean la etiqueta `app: inventario-app`.
* Permite que el tráfico externo acceda a los Pods balanceando la carga entre las réplicas.

#### 3. `k8s/secret.yaml`
Define un recurso tipo `Secret` de Kubernetes denominado `inventario-secret` con un par de datos de configuración de tipo `stringData`. Almacena la variable `API_KEY` para ser consumida de forma segura.

---

## 10. Despliegue

Siga esta secuencia de comandos para realizar el despliegue en un clúster local de Kubernetes (Minikube):

### 1. Iniciar Minikube
Asegúrese de tener Minikube activo en su terminal local:
```bash
minikube start
```

### 2. Aplicar el Secret de configuración
Cree los secretos requeridos por los contenedores antes de iniciar la carga de trabajo:
```bash
kubectl apply -f k8s/secret.yaml
```

### 3. Aplicar el Deployment base
Instancie los Pods de la aplicación:
```bash
kubectl apply -f k8s/deployment.yaml
```

### 4. Aplicar el Service
Exponga la aplicación a la red:
```bash
kubectl apply -f k8s/service.yaml
```

### 5. Monitorear el despliegue progresivo
Espere a que Kubernetes levante todos los Pods y las sondas de preparación marquen las instancias como listas:
```bash
kubectl rollout status deployment/inventario-app
```

### 6. Inspeccionar el estado de los recursos en el clúster
* **Ver Pods creados:**
  ```bash
  kubectl get pods -l app=inventario-app
  ```
* **Ver estado del Deployment:**
  ```bash
  kubectl get deployments
  ```
* **Ver estado del Service:**
  ```bash
  kubectl get svc inventario-service
  ```

### 7. Acceder al servicio desde Minikube
Minikube no expone automáticamente las IPs de los servicios tipo NodePort a la red física de Windows. Use el siguiente comando para generar un puente y abrir el navegador local de forma directa:
```bash
minikube service inventario-service
```
O para obtener únicamente la URL en la terminal:
```bash
minikube service inventario-service --url
```

### 8. Verificar conectividad por Curl
```bash
curl <minikube_url>/health
```

---

## 11. Estrategia Blue-Green

La estrategia de despliegue **Blue-Green** se utiliza para mitigar los riesgos de inactividad durante las actualizaciones críticas.

### Por qué se eligió Blue-Green
1. **Tiempo de inactividad cero (Zero Downtime):** El cambio de versión ocurre a nivel de enrutamiento de red de forma instantánea al modificar el selector del Service.
2. **Validación previa y segura:** La nueva versión de la aplicación puede probarse y validarse en un entorno idéntico de producción (el clúster) antes de abrirla al público general.
3. **Rollback instantáneo:** Si la nueva versión presenta fallos tras ser publicada, el tráfico puede regresarse inmediatamente a la versión anterior cambiando de nuevo la etiqueta del selector del Service.

### Cómo funciona en el proyecto

* Se definen dos Deployments aislados: `inventario-blue` (versión v1, color azul) e `inventario-green` (versión v2, color verde).
* Ambos Deployments cargan la imagen desde el repositorio, pero se inyectan variables de entorno diferentes (`APP_VERSION` y `APP_COLOR`) para poder identificar visualmente cuál versión responde.
* Existe un único Service llamado `inventario-blue-green` (`k8s/blue-green/service.yaml`). Este servicio expone el puerto 80 y utiliza un selector basado en la etiqueta `color`.

### Diagrama ASCII de Conmutación

```
                        [ Petición del Cliente ]
                                   │
                                   ▼
                   [ Service: inventario-blue-green ]
                 ┌──────────────────────────────────┐
                 │  selector:                       │
                 │    app: inventario-app           │
                 │    color: green  (Tráfico actual)│
                 └─────────────────┬────────────────┘
                                   │
                ┌──────────────────┴──────────────────┐
                │                                     │
                ▼ (No recibe tráfico)                 ▼ (Recibe todo el tráfico)
       [ Pods: Blue (v1) ]                   [ Pods: Green (v2) ]
       ├── app: inventario-app               ├── app: inventario-app
       └── color: blue                       └── color: green
```

Para realizar la conmutación a la versión **Blue (v1)**, se cambia el selector en el Service:

```
                        [ Petición del Cliente ]
                                   │
                                   ▼
                   [ Service: inventario-blue-green ]
                 ┌──────────────────────────────────┐
                 │  selector:                       │
                 │    app: inventario-app           │
                 │    color: blue   (Tráfico nuevo) │
                 └─────────────────┬────────────────┘
                                   │
                ┌──────────────────┴──────────────────┐
                │                                     │
                ▼ (Recibe todo el tráfico)            ▼ (No recibe tráfico)
       [ Pods: Blue (v1) ]                   [ Pods: Green (v2) ]
       ├── app: inventario-app               ├── app: inventario-app
       └── color: blue                       └── color: green
```

### Comandos reales para implementar la estrategia

1. **Desplegar ambas versiones simultáneamente en el clúster:**
   ```bash
   kubectl apply -f k8s/blue-green/blue.yaml
   kubectl apply -f k8s/blue-green/green.yaml
   ```

2. **Crear el servicio apuntando por defecto a Green (configuración inicial del manifiesto):**
   ```bash
   kubectl apply -f k8s/blue-green/service.yaml
   ```

3. **Validar que el servicio responde en la versión Green:**
   Abra el servicio en el navegador y valide que el color del banner sea verde:
   ```bash
   minikube service inventario-blue-green
   ```

4. **Conmutar tráfico a la versión Blue de forma instantánea:**
   Aplique un parche en caliente al selector del servicio para cambiar la etiqueta `color` a `blue`:
   ```bash
   kubectl patch svc inventario-blue-green -p '{"spec":{"selector":{"color":"blue"}}}'
   ```

5. **Verificar el rollback (si se detectan errores en Blue, regresar a Green):**
   ```bash
   kubectl patch svc inventario-blue-green -p '{"spec":{"selector":{"color":"green"}}}'
   ```

### Ventajas y Limitaciones de Blue-Green

#### Ventajas
* Aislamiento completo de los entornos durante la fase de prueba.
* Conmutación y retorno rápidos sin necesidad de recrear Pods en caliente.
* Cero tiempo de inactividad visible para los clientes.

#### Limitaciones
* Duplicación del consumo de recursos de cómputo en el clúster al mantener activos ambos entornos (`blue` y `green`).
* La consistencia de los datos locales se rompe entre las versiones debido a la falta de persistencia compartida.

---

## 12. Secret de Kubernetes

El clúster utiliza un secreto de tipo `Opaque` llamado `inventario-secret` (creado desde `k8s/secret.yaml`).

### Cómo se consume
Los Deployments (`blue.yaml` y `green.yaml`) inyectan este secreto en los Pods como una variable de entorno mediante la especificación `secretKeyRef`:
```yaml
env:
  - name: API_KEY
    valueFrom:
      secretKeyRef:
        name: inventario-secret
        key: API_KEY
```
En el backend Node.js, esta variable se lee mediante `process.env.API_KEY` y se valida su carga exitosa en la ruta `/config`.

### Por qué nunca debe almacenarse en Git
* **Seguridad:** Los secretos incluidos de forma explícita en archivos planos dentro de repositorios Git son fácilmente indexables y vulnerables a filtraciones.
* **Administración de Entornos:** Los secretos de Kubernetes permiten inyectar credenciales correspondientes a cada entorno (Desarrollo, Staging, Producción) sin alterar la base de código del software.
* *Nota:* En un entorno productivo real, la definición YAML del secreto con la clave codificada en base64 no debe subirse al control de versiones; en su lugar, se utilizan gestores externos de secretos (como Sealed Secrets, HashiCorp Vault o AWS Secrets Manager).

---

## 13. Escaneo de seguridad

El pipeline de integración continua incorpora un escaneo estático de la imagen Docker mediante **Trivy** (`aquasecurity/trivy-action@v0.36.0`).

### Funcionamiento y Configuración
* **Foco del escaneo:** Analiza las dependencias del sistema operativo (`vuln-type: os`) de la imagen final generada.
* **Severidad:** Filtra y busca vulnerabilidades únicamente clasificadas como críticas (`severity: CRITICAL`).
* **Exit Code (`exit-code: "1"`):** Si Trivy detecta una sola vulnerabilidad crítica no solucionada en la capa de sistema operativo, finaliza el proceso de escaneo enviando un código de error de sistema `1`. Esto detiene la ejecución del Job e impide que el pipeline ejecute los pasos siguientes (`docker push`), protegiendo el entorno de producción de imágenes comprometidas.

---

## 14. Persistencia de datos

La aplicación utiliza un almacenamiento basado en un archivo local JSON (`data/products.json`). 

### Comportamiento observado
Al eliminar o reiniciar un Pod de Kubernetes (por ejemplo, al ejecutar un comando manual `kubectl delete pod` o durante un proceso de Rolling Update), cualquier producto creado o eliminado por el usuario desaparece, y el catálogo se reinicia a su estado inicial de semilla.

### Causa técnica
Los contenedores son efímeros y *stateless* por definición. Al escribir en `data/products.json`, los cambios se guardan únicamente dentro de la capa de lectura y escritura del sistema de archivos local del Pod. Cuando el Pod es destruido, esa capa se elimina permanentemente.

*Nota:* Este comportamiento es el esperado y diseñado para la presente práctica académica del laboratorio de despliegues y CI/CD.

---



## 17. Conclusiones

La implementación del flujo de integración continua por medio de GitHub Actions asegura que el código sea testeado bajo un entorno consistente de manera previa a su compilación. El uso de validaciones automáticas como `npm test` previene que errores de lógica o regresiones críticas afecten la estabilidad del código en producción, garantizando la entrega de software de calidad.

La arquitectura de construcción multietapa en Docker representa una optimización de diseño de infraestructura clave para entornos de producción. Separar las herramientas de compilación de las de tiempo de ejecución reduce significativamente el peso de la imagen final y bloquea vectores de ataque potenciales al excluir las suites de pruebas y dependencias del entorno de desarrollo.

El despliegue sobre Kubernetes administrado localmente con Minikube introduce capacidades nativas de tolerancia a fallos. El uso de sondas de salud garantiza una gestión reactiva de las instancias del contenedor, de modo que el clúster puede redirigir las peticiones del usuario exclusivamente a los pods que se encuentren listos y reiniciar automáticamente los contenedores inactivos o con fallos internos.

Finalmente, la estrategia de despliegue Blue-Green y la inyección de configuraciones a través de Kubernetes Secrets establecen las pautas para operaciones de grado empresarial. La conmutación rápida de tráfico a nivel de balanceador de carga no solo elimina el tiempo de inactividad del servicio, sino que proporciona un mecanismo de rollback inmediato frente a incidentes inesperados en producción.

---
