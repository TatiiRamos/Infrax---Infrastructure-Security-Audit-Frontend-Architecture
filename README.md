# Infrax — Network Security Audit & Frontend Architecture

![Security Standard](https://img.shields.io/badge/Security-Network%20Audit-red)

![Frontend](https://img.shields.io/badge/Frontend-React%20%7C%20TypeScript-blue)

![Vite](https://img.shields.io/badge/Vite-5.4.21-purple)

![License](https://img.shields.io/badge/Architecture-Multi--Tenant-green)



Este repositorio documenta la auditoría de seguridad de infraestructura de red y el desarrollo del flujo de interfaz frontend para la plataforma SaaS multi-tenant **Infrax**.


El proyecto abarca la resolución de restricciones de acceso entre entornos virtualizados aislados, el análisis de tráfico TCP en la capa de transporte y la optimización de la experiencia de usuario (UX) en componentes críticos de autenticación.



---



## 🛠️ Tech Stack & Herramientas



* **Security & Network Analysis:** Kali Linux, Nmap, Wireshark, cURL.

* **Frontend Architecture:** React 18, TypeScript, Vite, Tailwind CSS, Firebase Auth.

* **Virtualization Environment:** Oracle VirtualBox (Host-Only Network), WSL2.



---



## 🛡️ Auditoría de Infraestructura y Redes



### 1. Topología del Entorno de Pruebas

Se configuró un entorno virtualizado aislado para simular escenarios de auditoría interna y evaluar la superficie de exposición del servidor web de desarrollo:



* **Host / Servidor de Desarrollo:** Windows (WSL2) — `192.168.56.102`

* **Auditor Node (VM):** Kali Linux — `192.168.56.101`

* **Segmento de Red:** VirtualBox Host-Only Ethernet Adapter

* **Servicio Auditado:** Vite Web Server en puerto `5173/tcp`



---



### 2. Diagnóstico de Remediación HTTP 403 (DNS Rebinding)



* **Incidente:** Al intentar acceder al servidor local desde la máquina de auditoría (Kali Linux), Vite rechazaba la conexión retornando un estado `HTTP 403 Forbidden` (`Blocked request`).

* **Causa Raíz:** Protección nativa de Vite contra ataques de DNS Rebinding que restringe las peticiones cuyo encabezado `Host` no coincide con las interfaces autorizadas.

* **Solución Implementada:** Reconfiguración del servidor en `vite.config.ts` para habilitar el binding global (`0.0.0.0`) y autorizar el filtrado de hosts en la red interna:



```typescript

// vite.config.ts

import { defineConfig } from 'vite';

import react from '@vitejs/plugin-react';



export default defineConfig({

  plugins: [react()],

  server: {

    host: '0.0.0.0',

    allowedHosts: true

  }

});



📷 Evidencias de Validación Técnica

🔹 Fingerprinting y Escaneo de Puertos (Nmap)

Verificación del estado del puerto y huella digital del servicio desde el nodo Kali Linux:

<img width="1366" height="708" alt="Captura de pantalla (5149)" src="https://github.com/user-attachments/assets/2cb25318-3b39-40bf-a136-803ae62550f1" />



Bash

nmap -Pn -sV -p 5173 192.168.56.102 -oA docs/evidencias/evidencia_puerto_infrax

Resultado: Puerto 5173/tcp en estado OPEN. Servicio reconocido entregando el punto de entrada de la aplicación.



🔹 Inspección del Handshake HTTP (cURL)

Validación del renderizado del DOM y retorno de cabeceras HTTP desde la terminal remota:

<img width="1366" height="705" alt="Captura de pantalla (5162)" src="https://github.com/user-attachments/assets/0d8a5a3f-5626-45fe-b8ac-53f43f790ad2" />



Bash

curl [http://192.168.56.102:5173](http://192.168.56.102:5173)

Resultado: Estado HTTP 200 OK con entrega completa del marcado HTML (<div id="root"></div>).



🔹 Captura de Tráfico en Capa de Red (Wireshark)

Captura y análisis de paquetes TCP/IP durante el intercambio de peticiones entre el Host y el cliente virtualizado.



Evidencia Binaria: Se adjunta el archivo de captura nativo trafico_infrax.pcapng para análisis forense de red.

<img width="1366" height="695" alt="Captura de pantalla (5161)" src="https://github.com/user-attachments/assets/32693a05-2f29-4b70-b1e9-cf4f42f081a5" />





💻 Desarrollo Frontend & Optimización UX

Contribuciones enfocadas en la resiliencia y accesibilidad de la interfaz de usuario:



Mapeo de Excepciones de Autenticación: Localización al español de las respuestas de error de Firebase Auth (auth/invalid-credential, auth/user-not-found).



Estados de Carga: Implementación de feedback visual (spinners y estados deshabilitados) en AuthForm.tsx para evitar double-submit.



PWA & Web Metadata: Configuración de manifest.json, viewport dinámico y favicon para soporte PWA.



## ⚙️ Arquitectura Frontend, Emuladores y Control de Acceso (RBAC)



### 1. Entorno Local de Pruebas (Firebase Emulator Suite)

Para garantizar pruebas aisladas y deterministas sin depender de servicios en la nube, la plataforma utiliza el emulador local de Firebase para servicios de **Auth** y **Firestore Database**:



* **Persistencia de Datos (`./emulator-seed`):** Mantiene el estado local de prueba grabado en disco (usuarios de test y permisos del sistema) para evitar la pérdida de sesión entre reinicios.

* **Carga Automática de Semilla:** Integración de scripts dedicados (`seed.js` / `seed-roles.mjs`) para poblar la base de datos local con la jerarquía base del tenant.



### 2. Flujo de Autenticación y Resolución de Permisos

El sistema implementa un modelo de control de acceso basado en roles (**RBAC**) en un entorno SaaS multi-tenant:



* **Sincronización de Sesión (`auth/initAuth`):** Escucha proactiva mediante `onAuthStateChanged` para validar el estado de la sesión activa en tiempo real.

* **Resolución Dinámica de Roles:** Evaluación de metadatos en Firestore (`tenants/dev/roles/geoadmin`) verificando el estado del perfil (`status: approved`) antes de autorizar el renderizado de la interfaz.

* **Gestión de Caching & Service Worker:** Control y prevención de inconsistencias de sesión provocadas por respuestas cacheadas en `sw.js` durante el ciclo de vida de desarrollo local.



---



## 🚀 Guía de Ejecución Local



### Paso 1: Levantar Servidores de Emulación (Terminal 1)

```bash

npm run dev:emulator

Inicia los emuladores de Firebase Auth y Firestore importando automáticamente la semilla desde ./emulator-seed.



Paso 2: Iniciar Servidor de Desarrollo Frontend (Terminal 2)

Bash

npm run dev

Inicia Vite en http://localhost:5173 habilitando el acceso dinámico.



---



## 📸 Evidencias de Desarrollo Frontend & RBAC



### 🔹 Resolución de Rol y Dashboard Principal

Validación del flujo de autenticación, hidratación de perfil (`Tatii Ramos`) y carga dinámica del panel con permisos `geoadmin`:

<img width="1309" height="679" alt="Captura de pantalla (5221)" src="https://github.com/user-attachments/assets/5e5449e1-3fd5-4aaa-a92d-df318b6c4827" />



* **Usuario Autenticado:** `tatiiramos9@gmail.com`

* **Estado de Cuenta:** `approved`

* **Rol Activo:** `geoadmin`

* **Tenant Seleccionado:** `DEV TENANT` (`SMLXL · DEV`)

* **Módulos Validados:** Acceso completo a *Tenants*, *Métricas Sistema*, *Métricas Global* y barra de navegación flotante.



📁 Estructura del Repositorio

Plaintext

├── docs/

│   └── evidencias/         # Capturas (.png), huellas (.nmap) y paquetes (.pcapng)

├── src/

│   ├── components/         # Componentes UI (Auth, Layout, Spinners)

│   └── config/             # Configuración de servicios y Firebase

├── vite.config.ts          # Configuración de servidor y red

└── README.md               # Documentación principal 
