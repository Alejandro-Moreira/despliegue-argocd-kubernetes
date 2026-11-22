# Despliegue de una aplicación con Argo CD y Kubernetes (GitOps)

## 1. Descripción del proyecto
Este proyecto implementa el despliegue automatizado de una aplicación web en un clúster Kubernetes utilizando Docker, Kind y Argo CD bajo el enfoque GitOps.
Se utiliza un repositorio público de GitHub como fuente de verdad para almacenar los manifiestos de Kubernetes. Argo CD se encarga de supervisar el repositorio y sincronizar los cambios con el clúster mediante la política de sincronización automática (Automated Sync).
La aplicación web utilizada para esta implementación es Nginx.

## 2. Objetivos
- Implementar un clúster Kubernetes utilizando Kind sobre Docker.
- Desplegar una aplicación web mediante un Deployment.
- Configurar un Service de tipo NodePort para acceder a la aplicación.
- Implementar escalamiento horizontal mediante Horizontal Pod Autoscaler (HPA).
- Configurar Metrics Server para obtener métricas de CPU y memoria.
- Integrar Kubernetes con GitHub mediante Argo CD.
- Automatizar la sincronización de los recursos utilizando GitOps.

## 3. Tecnologías utilizadas

| Tecnología | Descripción |
|---|---|
| Docker Desktop | Entorno para ejecutar contenedores. |
| Kubernetes | Orquestación de contenedores. |
| Kind | Creación de un clúster Kubernetes local. |
| kubectl | Administración del clúster. |
| Git | Control de versiones. |
| GitHub | Repositorio de manifiestos. |
| Argo CD | Automatización de despliegues GitOps. |
| Metrics Server | Recolección de métricas del clúster. |
| Nginx | Aplicación web de prueba. |

## 4. Arquitectura del proyecto
El funcionamiento de la implementación sigue este flujo:

1. Los manifiestos YAML se almacenan en GitHub.
2. Argo CD supervisa el repositorio.
3. Cuando se detecta un cambio, Argo CD inicia la sincronización automática.
4. Kubernetes aplica los manifiestos y mantiene los recursos declarados.
5. El HPA supervisa el consumo de CPU y ajusta el número de réplicas cuando corresponde.
6. El Service permite acceder a la aplicación web mediante NodePort.

## 5. Estructura del repositorio

```text
despliegue-argocd-kubernetes/
│
├── README.md
├── kind-config.yaml
│
└── k8s/
    ├── deployment.yml
    ├── service.yml
    └── hpa.yml
```

## 6. Configuración de Kubernetes

### 6.1. Deployment
Archivo: `k8s/deployment.yml`
Se configura un Deployment denominado `web-app` utilizando la imagen `nginx:1.27`.
Características:

- Dos réplicas iniciales.
- Etiquetas y selectores para identificar los Pods.
- Puerto del contenedor: 80.
- Solicitud de CPU: 100m.
- Límite de CPU: 500m.
- Solicitud de memoria: 64Mi.
- Límite de memoria: 256Mi.

### 6.2. Service
Archivo: `k8s/service.yml`
Se configura un servicio denominado `web-service` con las siguientes características:

- Tipo: NodePort.
- Puerto del servicio: 80.
- Puerto de destino: 80.
- NodePort: 30080.
- Selector: `app: web-app`.

La aplicación puede consultarse localmente mediante:

http://localhost:30080

### 6.3. Horizontal Pod Autoscaler (HPA)

Archivo: `k8s/hpa.yml`
Se configura el escalamiento horizontal automático mediante `HorizontalPodAutoscaler`.

Parámetros:

| Parámetro | Configuración |
|---|---|
| Deployment asociado | web-app |
| Réplicas mínimas | 2 |
| Réplicas máximas | 5 |
| Métrica | CPU |
| Utilización objetivo | 50% |
| API | autoscaling/v2 |

El HPA permite ajustar automáticamente el número de Pods según la utilización promedio de CPU.

## 7. Metrics Server
Se instala Metrics Server en el namespace `kube-system` para proporcionar las métricas necesarias para el funcionamiento del HPA.
En el entorno local de Kind se configura el argumento `--kubelet-insecure-tls` para facilitar la comunicación con Kubelet durante las pruebas del laboratorio.
Esta configuración es exclusiva para el entorno de pruebas y no se recomienda para producción.

## 8. Integración con Argo CD

Argo CD se instala en el clúster Kubernetes dentro del namespace `argocd`.

Posteriormente, se configura una aplicación con los siguientes parámetros:

| Parámetro | Valor |
|---|---|
| Nombre | biblioteca-web |
| Repositorio | GitHub |
| Rama | main |
| Ruta | k8s |
| Proyecto | default |
| Clúster destino | https://kubernetes.default.svc |
| Namespace destino | default |
| Política de sincronización | Automated Sync |

Argo CD utiliza los manifiestos del repositorio como referencia para mantener sincronizados los recursos de Kubernetes.

## 9. Pruebas de funcionamiento
Para comprobar la implementación se consideran las siguientes verificaciones:

### Estado del clúster
```bash
kubectl get nodes
```

### Estado de los Pods
```bash
kubectl get pods
```

### Estado del Deployment
```bash
kubectl get deployments
```

### Estado del Service
```bash
kubectl get services
```

### Estado del HPA
```bash
kubectl get hpa
```

### Métricas de Kubernetes
```bash
kubectl top nodes
kubectl top pods -A
```

### Acceso a la aplicación
http://localhost:30080

### Verificación de GitOps
Se modifica un manifiesto en el repositorio GitHub y se envían los cambios mediante Git. Argo CD detecta las modificaciones y realiza la sincronización automática con Kubernetes.

## 10. Resultados esperados
- Clúster Kubernetes local funcionando mediante Kind.
- Aplicación Nginx desplegada en Kubernetes.
- Dos réplicas iniciales de la aplicación.
- Servicio NodePort accesible desde el navegador.
- HPA configurado con un mínimo de dos y un máximo de cinco réplicas.
- Metrics Server proporcionando métricas al clúster.
- Argo CD conectado al repositorio público de GitHub.
- Sincronización automática de los manifiestos mediante GitOps.

## 11. Autor
**Alejandro Moreira**