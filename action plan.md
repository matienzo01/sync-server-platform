# Plan de Acción: Sincronización Discord-LuckPerms

Este documento detalla la estrategia y los pasos para construir una plataforma robusta que sincronice roles de Discord con permisos de LuckPerms.

## Estrategia Arquitectónica

Se evaluaron dos enfoques principales para la arquitectura multi-tenant.

### 1. Acceso Directo a la Base de Datos del Cliente (Enfoque Rechazado)

*   **Descripción:** Cada cliente nos proporcionaría credenciales para acceder directamente a su base de datos de LuckPerms.
*   **Decisión:** Este modelo ha sido **completamente descartado**. Aunque parece simple, presenta riesgos inaceptables.
*   **Razones para el Rechazo:**
    *   **Pesadilla de Seguridad:** Nos obligaría a almacenar credenciales de bases de datos de producción de clientes, convirtiéndonos en un objetivo de alto valor para atacantes. Una brecha en nuestro sistema expondría a todos nuestros clientes.
    *   **Infierno Operacional:** Sería imposible dar soporte a múltiples versiones de LuckPerms, gestionar la conectividad a cientos de bases de datos distintas y diagnosticar problemas de fiabilidad que podrían originarse en la infraestructura del cliente.
    *   **Nula Escalabilidad:** El modelo de una conexión de base de datos por tenant es ineficiente y no escala.

### 2. API Centralizada con Storage Provider Personalizado (Enfoque Elegido)

*   **Descripción:** Este es el estándar de la industria para aplicaciones SaaS. Los clientes instalarán una versión modificada de LuckPerms que, en lugar de conectarse a una base de datos, se comunicará con nuestra API central a través de una API Key.
*   **Decisión:** **Este es el camino a seguir.** Es seguro, escalable y mantenible a largo plazo.
*   **Ventajas:**
    *   **Seguridad Robusta:** No se almacenan credenciales de clientes. El acceso se controla mediante API Keys que pueden ser revocadas.
    *   **Control y Mantenimiento:** La lógica y los datos están centralizados, simplificando actualizaciones, backups y monitoreo.
    *   **Escalabilidad Real:** La arquitectura nos permite escalar nuestra API y base de datos de forma eficiente.

## Plan de Implementación por Fases

El desarrollo se dividirá en dos fases claras para mitigar riesgos y validar la lógica de negocio antes de abordar la parte más compleja.

### Fase 1: Prueba de Concepto (PoC) - Prioridad Inmediata

**Objetivo:** Validar la lógica de sincronización de extremo a extremo en un entorno controlado, sin modificar el núcleo de LuckPerms.

**Arquitectura de la PoC:**
Usaremos los componentes existentes (`bot-discord`, `rest-api`) y una base de datos, todo orquestado con `docker-compose`. El `bot-discord` se comunicará con la `rest-api` estándar de LuckPerms, que a su vez se conectará a la base de datos. Esto simula perfectamente el flujo de datos de la arquitectura final.

**Pasos:**

1.  **Crear un Servicio `LuckPermsService.ts` en `bot-discord`:**
    *   **Ubicación:** `bot-discord/src/services/LuckPermsService.ts`.
    *   **Responsabilidad:** Encapsular toda la comunicación con la `rest-api` de LuckPerms. Utilizará `axios` o similar.

2.  **Definir Interfaces Claras:**
    *   Crear interfaces TypeScript para los payloads y respuestas de la API de LuckPerms, asegurando la seguridad de tipos.

3.  **Implementar Métodos de la API:**
    *   Desarrollar los métodos principales en `LuckPermsService.ts` (`addPermission`, `promoteUser`, etc.), asegurándose de pasar el contexto del servidor (`server=<server_id>`) en cada llamada.

4.  **Configurar el Entorno:**
    *   Usar variables de entorno en `bot-discord` para definir la URL de la `rest-api`.

5.  **Integrar y Probar:**
    *   Inyectar `LuckPermsService` en la lógica del bot.
    *   Crear comandos o listeners de eventos en Discord (ej: al añadir un rol) que disparen las llamadas al servicio.
    *   Verificar que los permisos se aplican correctamente en la base de datos a través de la `rest-api`.

### Fase 2: Producto Mínimo Viable (MVP) - Objetivo a Largo Plazo

**Objetivo:** Construir la plataforma multi-tenant de producción, reemplazando el `storage-method` de LuckPerms.

**Prerrequisito:** La Fase 1 debe estar completada y validada.

**Pasos:**

1.  **Desarrollar el Módulo `rest-storage` para LuckPerms:**
    *   **Tarea Principal:** Crear un nuevo módulo en el proyecto `LuckPerms` que implemente la interfaz `StorageImplementation`.
    *   **Funcionalidad:** En lugar de código JDBC, este módulo hará llamadas HTTP a nuestra API central para cada operación de datos.

2.  **Evolucionar la `rest-api` a un Servicio Multi-Tenant:**
    *   Implementar un sistema de autenticación robusto basado en API Keys.
    *   Añadir lógica de autorización para que una clave de un tenant solo pueda acceder a los datos de ese tenant.

3.  **Adaptar `bot-discord`:**
    *   Modificar el `LuckPermsService` para que incluya la API Key correspondiente en las cabeceras de cada solicitud.

4.  **Empaquetado y Documentación:**
    *   Crear la distribución del plugin de LuckPerms modificado.
    *   Escribir documentación clara para que los clientes sepan cómo instalar el plugin y configurarlo con la API Key de su cuenta.
