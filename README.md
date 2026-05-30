# 🚀 Plataforma de Microservicios Contenedorizada con Arquitectura CI/CD en AWS

Este repositorio contiene la arquitectura, el código fuente y los pipelines de automatización para la plataforma de microservicios de la empresa **Innovatech Chile**. El proyecto implementa un ecosistema multinivel completamente contenedorizado, desacoplado y con un flujo automatizado de Integración y Despliegue Continuos (CI/CD) hacia la nube de Amazon Web Services (AWS).

---

## 🏗️ Arquitectura del Sistema

La solución se compone de 4 microservicios independientes, diseñados bajo buenas prácticas *Cloud-Native*:

1.  **Frontend (`front_despacho`):** Aplicación SPA desarrollada en React y Vite, servida de forma segura mediante un servidor Nginx optimizado.
2.  **API Ventas (`back-Ventas_SpringBoot`):** Microservicio de backend construido sobre Spring Boot 3.4.4 y Java 17, encargado de procesar transacciones comerciales.
3.  **API Despachos (`back-Despachos_SpringBoot`):** Microservicio de backend en Spring Boot que gestiona la logística de envíos.
4.  **Capa de Datos (`db`):** Motor de base de datos MySQL 8.0/MariaDB parametrizado para persistencia aislada de datos.

---

## 🛠️ Tecnologías Utilizadas

* **Contenedores y Orquestación:** Docker, Dockerfile (Multi-stage), Docker Compose.
* **Backend:** Java 17, Spring Boot 3.x, JPA / Hibernate.
* **Frontend:** React, Vite, Nginx.
* **Nube (AWS):** Amazon EC2, Amazon ECR (Elastic Container Registry), AWS Systems Manager (SSM), IAM Roles.
* **CI/CD:** GitHub Actions (Flujos en formato YAML).

---

## 🔒 Buenas Prácticas DevOps Implementadas

### 1. Optimización de Imágenes (Multi-stage Build)
Los backends implementan Dockerfiles multi-etapa. Se utiliza una imagen pesada con Maven para la compilación (`AS builder`) y se descarta en la fase de ejecución, copiando el artefacto `.jar` en una imagen base ligera `eclipse-temurin:17-jre-alpine`. Esto minimiza el tamaño de la imagen y reduce vulnerabilidades.

### 2. Principio de Mínimo Privilegio (Seguridad No-Root)
Por defecto, los servicios no corren como administrador. Se configuraron explícitamente usuarios del sistema sin privilegios (`appuser` y `nginx`) para ejecutar los procesos internos del contenedor.

### 3. Persistencia de Datos
El almacenamiento de la base de datos local utiliza un **Named Volume** (`db_data:/var/lib/mysql`) administrado de forma nativa por Docker, garantizando la persistencia y la inmunidad de los datos frente al ciclo de vida de los contenedores.

### 4. Despliegue Seguro sin Puertos Abiertos (DevSecOps)
El pipeline de Despliegue Continuo (CD) no utiliza SSH ni requiere exponer el puerto 22 de las instancias EC2 a internet. En su lugar, utiliza **AWS Systems Manager (SSM)** mediante el comando `aws ssm send-command`, ejecutando scripts remotos de forma segura a través de los roles nativos de IAM (`labRole`).

---

## 🚀 Flujo de CI/CD (GitHub Actions)

Los pipelines están automatizados mediante workflows individuales en `.github/workflows/`. Cada vez que se realiza un evento de `push` en la rama `deploy`, se ejecuta el siguiente proceso:

1.  **Filtro por Ruta (`paths`):** El selector optimiza recursos activando únicamente el pipeline del microservicio que sufrió cambios físicos.
2.  **Construcción e Inyección:** Se realiza el *checkout*, se autentican las credenciales seguras mediante *GitHub Secrets*, y en el Frontend se inyecta la IP Pública de AWS en tiempo de compilación (`build-time`).
3.  **Push a Registro:** La imagen inmutable se sube a su respectivo repositorio privado en **Amazon ECR**.
4.  **Despliegue Automatizado:** GitHub invoca a AWS SSM para ordenar a la instancia EC2 que descargue la nueva imagen (`docker pull`), detenga el contenedor obsoleto y ejecute el nuevo servicio con sus respectivas variables de entorno.

---

## 💻 Ejecución del Proyecto

### Requisitos Previos
* Docker Desktop e instalado.
* Git configurado.

### Entorno de Desarrollo Local
Para clonar, compilar y levantar de forma integrada todo el ecosistema local en menos de un minuto, ejecute en su terminal:

```bash
# Clonar repositorio y cambiar a la rama deploy
git clone [https://github.com/Manuxc-sys/Proyecto-DevOps-ev2.git](https://github.com/Manuxc-sys/Proyecto-DevOps-ev2.git)
cd Proyecto-DevOps-ev2
git checkout deploy

# Levantar el entorno orquestado con Docker Compose
docker compose up -d --build
