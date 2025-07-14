# Gemini Code Assistant Report

## Project Overview

The primary goal of this project is to create a robust system for synchronizing roles between a Discord server and a LuckPerms-managed game server environment. The synchronization is bidirectional, meaning that assigning a role in Discord will update the user's permissions in LuckPerms, and vice-versa.

To achieve this, the project is divided into three main components:

1.  **LuckPerms Core Engine (`/home/matias/repos/LuckPerms`):** The foundational permissions plugin. This component will require modifications to support "dynamic contexts" or a similar mechanism. This will allow permissions to be managed relative to different game servers within a single database, which is crucial for multi-server environments.

2.  **LuckPerms REST API (`/home/matias/repos/rest-api`):** An extension that provides a RESTful interface to interact with LuckPerms. This API will also need to be modified to expose the new dynamic context functionalities, allowing external services to manage permissions programmatically.

3.  **Synchronization Service (`/home/matias/repos/bot-discord`):** A backend service built with Node.js and TypeScript that acts as the central orchestrator. It connects to both the Discord API and the modified LuckPerms REST API to handle the synchronization logic. It will listen for events from both platforms (e.g., role changes in Discord, permission changes in LuckPerms) and trigger the corresponding updates on the other platform.

## Architectural Design

Two primary architectural models are being considered for the platform. The initial plan is to develop the simpler option first and evaluate the feasibility of the more complex model later.

### Option 1: Decentralized Model (The "Simple Option")

In this model, each consumer is responsible for hosting their own LuckPerms server and database.
-   **Mechanism:** Consumers provide their LuckPerms database credentials to our synchronization service.
-   **Our Role:** The service connects directly to each consumer's database on-demand to read or write permissions data.
-   **Pros:**
    -   Faster initial development and implementation.
-   **Cons:**
    -   **Security Risk:** Requires storing and managing sensitive database credentials for multiple tenants.
    -   **Complexity:** Introduces challenges in managing a multi-tenant architecture with numerous direct database connections.

### Option 2: Centralized API Model (The "Complex Option")

This model involves a centralized database managed by our platform, abstracting the data storage from the consumers.
-   **Mechanism:** This would require a significant modification to the LuckPerms core plugin to add a new "REST API" storage provider. Consumers would configure their game server's LuckPerms plugin with an API key provided by our platform.
-   **Our Role:** All data interactions from the consumer's server would be routed through our secure, centralized REST API, which then communicates with our database.
-   **Pros:**
    -   **Enhanced Security:** No need to store consumer database credentials; authentication is handled via revocable API keys.
    -   **Scalability & Control:** Centralized data management simplifies maintenance, backups, and feature rollouts.
-   **Cons:**
    -   **High Complexity:** Requires deep modification of the LuckPerms core, a complex and time-consuming task.

### Option 3: Centralized Database with Schema-per-Tenant Isolation (The "Hybrid Option")

This model serves as a powerful and secure intermediate, avoiding modifications to the LuckPerms core while still providing a centralized and scalable architecture. It uses a single database instance (e.g., PostgreSQL) with logical separation for each tenant.

-   **Mechanism:**
    1.  **Tenant Provisioning:** When a new consumer registers, our platform automatically creates a dedicated database **schema** (e.g., `tenant_123`) and a dedicated database **user** (e.g., `user_123`) with a unique password.
    2.  **LuckPerms Configuration:** The consumer configures their LuckPerms instance to connect to our central database using these credentials. The provided user has permissions *only* for its dedicated schema, ensuring data cannot be accessed by other tenants at the database level.
    3.  **Backend Service Access:** Our backend services (`rest-api`, `bot-discord`) connect to the database using a single, privileged **service account**. This account has read/write access to *all* tenant schemas.

-   **Our Role:** The application logic is responsible for ensuring that when acting on behalf of a tenant, it directs its queries to the correct schema (e.g., `SELECT * FROM tenant_123.luckperms_users;`).

-   **Pros:**
    -   **No LuckPerms Core Modification:** Leverages existing JDBC storage support in LuckPerms.
    -   **High Security:** Strong data isolation between tenants at the database level.
    -   **Simplified Backend:** Our services use a single, static database connection, simplifying configuration and improving connection pooling efficiency.
    -   **Centralized Management:** We retain control over the database for maintenance and backups.

-   **Cons:**
    -   **Application-Layer Complexity:** The responsibility for tenant data isolation shifts to our backend code, which must be carefully written to query the correct schemas.
    -   **Automated Provisioning Required:** Requires robust automation for creating and managing schemas and users for each tenant.

## Key Technologies

### Synchronization Service (bot-discord)
- **Backend:** Node.js, Express
- **Language:** TypeScript
- **Discord Interaction:** `discord.js`
- **Dependency Injection:** InversifyJS
- **Containerization:** Docker, Docker Compose
- **CI/CD:** GitHub Actions

### LuckPerms & REST API
- **Language:** Java
- **Build Tool:** Gradle
