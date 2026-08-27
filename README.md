# Infrax — Network Security Audit & Frontend Architecture

Este repositorio documenta la auditoría de seguridad de infraestructura de red y el desarrollo del flujo de interfaz frontend para la plataforma SaaS multi-tenant Infrax.
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

### 2. Diagnóstico de Remediación HTTP 403 (DNS Rebinding)

* **Incidente:** Al intentar acceder al servidor local desde la máquina de auditoría (Kali Linux), Vite rechazaba la conexión retornando un estado **HTTP 403 Forbidden** (*Blocked request*).
* **Causa Raíz:** Protección nativa de Vite contra ataques de DNS Rebinding, la cual restringe las peticiones cuyo encabezado `Host` no coincide con las interfaces autorizadas.
* **Solución Aplicada para Auditoría:** Reconfiguración en `vite.config.ts` para habilitar el binding en todas las interfaces de red (`0.0.0.0`) y autorizar las peticiones del segmento interno:

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    host: '0.0.0.0',      // Expone el servidor a la red local/virtualizada
    allowedHosts: true,   // Permite peticiones con headers de Host externos
  },
})
