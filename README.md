#  Proyecto Tienda Perritos - Innovatech Chile

Este repositorio contiene el código fuente, los manifiestos de infraestructura y la configuración de Integración y Entrega Continua (CI/CD) para el proyecto "Tienda Perritos". 

La aplicación está construida bajo una arquitectura de microservicios y se encuentra completamente contenedorizada para su despliegue orquestado en la nube de Amazon Web Services (AWS).

##  Arquitectura del Sistema

El proyecto está dividido en las siguientes capas:

* **Frontend:** Desarrollado con React (Vite), proporcionando una interfaz de usuario rápida y dinámica (Dashboard de Despachos y Ventas).
* **Backend:** Desarrollado con Spring Boot (Java), exponiendo una API RESTful para la gestión de la lógica de negocio.
* **Base de Datos:** Motor relacional (MySQL) para la persistencia de datos.
* **Infraestructura Cloud:** Desplegado en un clúster de **Amazon EKS** (Elastic Kubernetes Service). La topología de red se diseñó utilizando **2 tipos de subredes** (públicas y privadas) para garantizar la seguridad de los nodos de trabajo mientras se expone el frontend a través de un Application Load Balancer.

##  Pipeline CI/CD (GitHub Actions)

Este repositorio cuenta con un flujo de trabajo automatizado que se activa con cada `push` a la rama `main`. El pipeline realiza las siguientes etapas de forma autónoma:

1. **Checkout:** Clona el código fuente del repositorio.
2. **AWS Auth:** Autenticación segura con AWS Academy mediante credenciales temporales.
3. **Build & Push:** Construcción de las imágenes Docker (Frontend y Backend) y publicación automática en **Amazon ECR**.
4. **Deploy:** Actualización del *kubeconfig* y aplicación de los manifiestos YAML en el clúster EKS mediante `kubectl apply`.
5. **Rollout:** Reinicio automático de los *Deployments* (`imagePullPolicy: Always`) para asegurar que los pods descarguen y ejecuten la última versión en producción sin tiempo de inactividad.

##  Requisitos Previos

Para interactuar con este proyecto de forma local o modificar la infraestructura, es necesario contar con:

* [Docker Desktop](https://www.docker.com/)
* [AWS CLI](https://aws.amazon.com/es/cli/) configurado con credenciales activas.
* [kubectl](https://kubernetes.io/docs/tasks/tools/) para la gestión del clúster.
* Cuenta activa de AWS Academy (Learner Lab).

##  Configuración de Entorno (Secrets)

Para que el pipeline de GitHub Actions funcione correctamente, es imperativo configurar los siguientes *Secrets* en el repositorio (`Settings > Secrets and variables > Actions`):

* `AWS_ACCESS_KEY_ID`
* `AWS_SECRET_ACCESS_KEY`
* `AWS_SESSION_TOKEN` (Debe actualizarse cada vez que se inicie una nueva sesión en Learner Lab)
* `AWS_REGION` (ej. `us-east-1`)
* `EKS_CLUSTER_NAME`
* `EKS_NAMESPACE` (ej. `tienda`)
* `ECR_FRONTEND` (URI del repositorio de ECR para el frontend)
* `ECR_BACKEND` (URI del repositorio de ECR para el backend)

##  Despliegue Manual y Comprobación

Una vez que el pipeline finalice con éxito, puedes comprobar el estado de los recursos en el clúster ejecutando:

```bash
# Verificar el estado de los Pods
kubectl get pods -n tienda

# Obtener la URL (External-IP) del LoadBalancer para acceder a la aplicación
kubectl get svc -n tienda
