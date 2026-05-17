# Proyecto de Despliegue Automatizado y Contenedorización - Innovatech Chile (Etapa 2)

Este repositorio contiene la solución de arquitectura, contenedorización y automatización CI/CD para la plataforma de retail y monitoreo de la empresa **Innovatech Chile**. El sistema está diseñado bajo una arquitectura de microservicios robusta, desacoplada y escalable, desplegada sobre la infraestructura en la nube de **Amazon Web Services (AWS)**.

---

##  1. Arquitectura de Infraestructura en AWS
El despliegue operativo de la solución respeta estrictamente los principios de seguridad de red distribuidos en una **VPC (Virtual Private Cloud)** organizada en dos tipos de subredes:

* **Subred Pública:** Alberga la instancia **`EC2-INNOVATECH-FRONT`**. Expone un servidor inverso **Nginx** encargado de servir la aplicación de Frontend (React/Vite) y rutear de manera segura las peticiones hacia la capa de negocio. Es la única capa accesible directamente desde la Internet pública.
* **Subred Privada:** Aislada por completo del tráfico exterior para mitigar vectores de ataque. Contiene:
    * **`EC2-INNOVATECH-BACK`:** Ejecuta los microservicios de Spring Boot.
    * **`EC2-INNOVATECH-DATA`:** Ejecuta el contenedor del motor de base de datos MySQL.

---

##  2. Estructura del Ecosistema de Contenedores (Docker)
El stack tecnológico se encuentra completamente contenerizado mediante **Docker** y orquestado de forma modular a través de archivos **Docker Compose** independientes para garantizar la alta disponibilidad y la continuidad operativa:

### 🔹 Capa de Datos (`docker-compose.data.yml`)
* **Servicio:** `app-db-mysql` (Imagen Oficial `mysql:8.0`).
* **Puerto:** `3306` (Restringido localmente dentro de la subred privada).
* **Persistencia:** Implementada mediante **Named Volumes** para asegurar la persistencia del estado transaccional (ventas y despachos) ante reinicios críticos del contenedor.

### 🔹 Capa de Negocio (`docker-compose.back.yml`)
Levanta dos microservicios Java 17 / Spring Boot independientes que se comunican internamente con la base de datos:
* **Microservicio de Ventas (`app-back-ventas-1`):** Expone la lógica de negocio en el puerto mapeado `8081:8081`.
* **Microservicio de Despachos (`app-back-despacho-1`):** Diseñado originalmente sobre el puerto interno `8081`. Se implementó un mapeo espejo estratégico **`8082:8081`** para evitar colisiones de red a nivel de host, aislando la lógica de despachos de manera exitosa.

### 🔹 Capa de Presentación (`docker-compose.front.yml`)
* **Servicio:** Frontend web encapsulado que interactúa con las APIs inversas mediante variables de entorno optimizadas para el entorno productivo de AWS.

---

## ⚙️ 3. Pipeline de Integración y Despliegue Continuo (CI/CD)
Ubicado en la ruta `.github/workflows/`, el pipeline está automatizado mediante **GitHub Actions** y se gatilla exclusivamente al realizar un `push` o `merge` en la rama **`deploy`**.

### 🔄 Flujo del Workflow:
1.  **Build:** Compilación automatizada de los artefactos y construcción de las imágenes Docker utilizando configuraciones eficientes para reducir el tamaño de las capas.
2.  **Push:** Publicación automatizada de las imágenes en el registro de contenedores (Docker Hub / ECR).
3.  **Deploy:** Conexión automatizada vía SSH hacia las instancias correspondientes en AWS EC2, deteniendo los servicios anteriores, aplicando un `pull` de las nuevas versiones y levantando el stack actualizado con `docker compose up -d --force-recreate`.
4.  **Seguridad:** Toda la automatización se gestiona de forma segura abstrayendo tokens, llaves SSH y credenciales de AWS mediante **GitHub Secrets**.

---

## 🛠️ 4. Instrucciones para Ejecución Local / Manual

Si se requiere levantar o depurar la suite de servicios manualmente en las instancias, sitúese en la raíz del proyecto y ejecute los siguientes comandos según el componente:

**Para levantar la capa de datos (Instancia DATA):**
```bash
sudo docker compose -f docker-compose.data.yml up -d

Para reconstruir y levantar la lógica de negocio sin caché (Instancia BACK):
```bash

sudo docker compose -f docker-compose.back.yml down
sudo docker builder prune -a -f
sudo docker compose -f docker-compose.back.yml up -d --build
```
# Guardar cambios en github

git add .
git commit -m "fix: ajuste visual para demostración en vivo"
git push origin deploy


## 5. Procedimiento de Recertificación de Infraestructura (Cambio de IPs)

Dado que el entorno de laboratorio utiliza direccionamiento IP dinámico en AWS Academy, ante un reinicio completo de las instancias EC2 se debe ejecutar el siguiente protocolo de actualización en los GitHub Secrets del repositorio:

1. **Actualización de Credenciales de Host (CD):**
   * Modificar `EC2_HOST` con la nueva IP pública de la instancia de Frontend.
   * Modificar `EC2_BACK_HOST` con la nueva IP pública de la instancia de Backend.

2. **Actualización del Enlace del Cliente (Vite):**
   * Modificar `VITE_API_URL` utilizando la estructura `http://<NUEVA_IP_BACKEND>:8081` para actualizar el punto de enlace (endpoint) global del ecosistema Frontend de React hacia los microservicios.

3. **Gatillar Redespliegue:**
   * Realizar un ajuste menor o empujar un commit a la rama `deploy` para forzar la ejecución del pipeline y actualizar los contenedores con las nuevas variables de entorno en producción.
