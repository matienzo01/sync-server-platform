# Gemini Code Assistant Report

## Project Overview

The primary goal of this project is to create a robust system for synchronizing roles between a Discord server and a LuckPerms-managed game server environment. The synchronization is bidirectional, meaning that assigning a role in Discord will update the user's permissions in LuckPerms, and vice-versa.

To achieve this, the project is divided into three main components:

1.  **LuckPerms Core Engine (`/home/matias/repos/LuckPerms`):** The foundational permissions plugin. This component will require modifications to support "dynamic contexts" or a similar mechanism. This will allow permissions to be managed relative to different game servers within a single database, which is crucial for multi-server environments.

2.  **LuckPerms REST API (`/home/matias/repos/rest-api`):** An extension that provides a RESTful interface to interact with LuckPerms. This API will also need to be modified to expose the new dynamic context functionalities, allowing external services to manage permissions programmatically.

3.  **Synchronization Service (`/home/matias/repos/bot-discord`):** A backend service built with Node.js and TypeScript that acts as the central orchestrator. It connects to both the Discord API and the modified LuckPerms REST API to handle the synchronization logic. It will listen for events from both platforms (e.g., role changes in Discord, permission changes in LuckPerms) and trigger the corresponding updates on the other platform.

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
