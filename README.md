# 🐾 Tienda Perritos - Infraestructura DevOps & Despliegue Automatizado

[![CI/CD Frontend](https://img.shields.io/badge/CI%2FCD-Frontend-success?logo=github)](https://github.com/Bryckson/tienda-perritos/actions)
[![CI/CD Backend](https://img.shields.io/badge/CI%2FCD-Backend-success?logo=github)](https://github.com/Bryckson/tienda-perritos/actions)
[![CI/CD DB](https://img.shields.io/badge/CI%2FCD-DB-success?logo=github)](https://github.com/Bryckson/tienda-perritos/actions)

Aplicación de e-commerce "Tienda Perritos" (proyecto Innovatech Chile) con arquitectura de microservicios. Este repositorio contiene el código fuente y toda la automatización DevOps para su integración y despliegue continuo (CI/CD) en Amazon Web Services (AWS).

---

## 🏛 1. Arquitectura y Políticas de Seguridad (AWS)
El proyecto se despliega de forma distribuida en **tres instancias EC2 de AWS**. Se implementó un diseño de red estricto basado en **Security Groups (SGs)** encadenados para garantizar el principio de mínimo privilegio: solo el Frontend es accesible desde Internet, aislando completamente la lógica de negocio y los datos.

*(Sugerencia: Reemplaza esta línea con la ruta real de tu imagen)*
`![Diagrama de Arquitectura](./docs/arquitectura.png)`

### Configuración de Security Groups:
- 🌐 **Frontend EC2 (`FRONT-SG`):** 
  - **Acceso:** Público (Internet).
  - **Reglas:** Permite tráfico entrante desde cualquier origen (`0.0.0.0/0`) a los puertos `80` y `8080` (HTTP), además del puerto `22` (SSH) para administración.
  
- 🔒 **Backend EC2 (`BACKENS-SG`):** 
  - **Acceso:** Restringido (Subred Privada lógica).
  - **Reglas:** **NO** posee acceso abierto a Internet. Solo permite tráfico entrante en el puerto `3001` (API) y `22` (SSH) **exclusivamente desde el Security Group del Frontend (`sg-028e70f7cb2d3044a`)**.

- 🗄️ **Base de Datos EC2 (`BD-SG`):** 
  - **Acceso:** Altamente Restringido.
  - **Reglas:** Solo permite tráfico entrante en el puerto `3306` (MySQL) **exclusivamente desde el Security Group del Backend (`sg-0d6c601cea573419f`)**.

### Aprovisionamiento Automatizado (User Data)
Para cumplir con el principio DevOps de minimizar la configuración manual, las tres instancias EC2 fueron aprovisionadas utilizando un script de **User Data** durante su lanzamiento. Este script asegura que la infraestructura nazca lista para operar, ejecutando automáticamente:
- Actualización del sistema operativo (`yum update -y`).
- Instalación de dependencias clave (**Docker** y **Git**).
- Configuración de permisos: Se agrega el `ec2-user` al grupo `docker` (`usermod -aG docker ec2-user`) para permitir la ejecución del pipeline sin necesidad de escalado de privilegios.
- Instalación del plugin oficial de **Docker Compose v2** directamente en el directorio del usuario.

Gracias a esto, las instancias están preparadas desde el segundo cero para recibir instrucciones de despliegue a través de AWS SSM.

---

## 🐳 2. Estrategia de Contenedorización (Docker)
Todo el ecosistema de la aplicación fue empaquetado utilizando Docker. Esto nos permite separar las responsabilidades (Frontend, Backend y Base de Datos) y asegurar que el código se ejecute de la misma manera en un entorno de desarrollo local que en la infraestructura de AWS.

- **Imágenes Base:** Se utilizaron imágenes estándar como Node.js y Nginx para empaquetar las aplicaciones de forma rápida y funcional.
- **Orquestación Local (`docker-compose.yml`):** El archivo en la raíz del proyecto permite levantar todo el stack de manera local con un solo comando. Este archivo gestiona la red interna, las variables de entorno y los volúmenes, facilitando las pruebas antes de subir cambios al repositorio.

---

## 💾 3. Persistencia de Datos
Para garantizar la continuidad operativa ante reinicios o actualizaciones automáticas, se implementaron **Volúmenes de Docker**.

- **Implementación:** Se utiliza un *Named Volume* (`dbdata`) montado en el contenedor de la Base de Datos.
- **Justificación técnica:** A diferencia de un *bind mount*, un *named volume* es administrado por Docker, lo que lo hace más seguro y portable. Esto asegura que si GitHub Actions actualiza la imagen de la base de datos (haciendo `stop` y `rm` del contenedor anterior), **los registros de los productos no se pierdan** al levantar la nueva versión del contenedor.

---

## 🚀 4. Integración y Despliegue Continuo (CI/CD)
La entrega del software está 100% automatizada mediante **GitHub Actions**, segmentada por capa mediante *path triggers*. 

*(Sugerencia: Reemplaza esta línea con la ruta real de tu imagen)*
`![Flujo CI/CD](./docs/flujo-cicd.png)`

### Flujo del Pipeline (Archivos YAML)
Contamos con 3 workflows independientes (`cicd-tienda-frontend.yml`, `cicd-tienda-backend.yml`, `cicd-tienda-db.yml`). El flujo es:

1. **Trigger:** `on: push` en la rama `main` (o `deploy`), condicionado a cambios exclusivos en su respectiva carpeta (ej. `paths: ['backend/**']`).
2. **Build & Push a Amazon ECR:** 
   - El *runner* de GitHub compila la imagen Docker.
   - **Justificación de ECR:** Se eligió **Amazon ECR** sobre Docker Hub por ser un registro 100% privado, con integración nativa a AWS IAM para un control de acceso seguro y de alta velocidad de descarga hacia las instancias EC2.
3. **Deploy Automático vía AWS SSM:** 
   - En lugar de utilizar conexiones SSH estáticas (que requieren abrir puertos y gestionar llaves), el pipeline utiliza `aws ssm send-command` para ejecutar instrucciones de forma segura dentro de la instancia EC2 remota.
   - La EC2 inicia sesión en ECR, hace un `docker pull` de la nueva imagen, detiene/elimina el contenedor antiguo y levanta el nuevo (`docker run -d`).

### 🔐 Seguridad y Gestión de Secrets
Ninguna credencial está expuesta en el código fuente. Utilizamos **GitHub Secrets** para inyectar en tiempo de ejecución:
- Credenciales AWS: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`, `AWS_REGION`.
- URLs del Registry: `ECR_REPO_URL_FRONTEND`, `ECR_REPO_URL_BACKEND`, `ECR_REPO_URL_DB`.
- IDs de Instancias: `EC2_FRONTEND_INSTANCE_ID`, `EC2_BACKEND_INSTANCE_ID`, `EC2_DB_INSTANCE_ID`.

---

## 🛠 5. Uso y Ejecución Local

Para levantar el entorno en tu máquina local para desarrollo:

1. **Clonar el repositorio:**
   ```bash
   git clone https://github.com/Bryckson/tienda-perritos.git
   cd tienda-perritos

2. **Construir y levantar los contenedores:**
   code
   ```bash
   docker-compose up -d --build
   
3. **Acceder a la aplicación localmente:**
   - **Frontend:** http://localhost:80 o http://localhost:8080
   - **Backend API:** http://localhost:3001
   - **Base de datos:** Expuesta internamente en el puerto TCP 3306
