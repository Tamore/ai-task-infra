# AI Task Platform - Architecture Document

## System Overview
The AI Task Platform is designed to handle up to 100,000 tasks per day, utilizing a modern, scalable MERN stack supplemented by a high-throughput Redis queue and Python-based asynchronous workers. The entire platform is containerized and orchestrated via Kubernetes using an Argo CD GitOps pipeline.

## Component Architecture
1. **Frontend (Next.js)**
   - Delivers a highly responsive, premium UI using Next.js and Tailwind CSS.
   - Communicates with the Backend API over REST.

2. **Backend (Node.js/Express)**
   - Handles JWT-based authentication, rate limiting, and request validation.
   - Pushes tasks to the Redis queue (`ai-tasks-queue`) instead of executing them synchronously, ensuring fast API response times.

3. **Background Worker (Python)**
   - Continuously listens to the Redis queue using blocking pop (`BRPOP`) for zero-latency task acquisition.
   - Executes the AI processing operations (e.g., text manipulation) and updates the status directly in MongoDB.
   - Designed to be stateless, allowing horizontal scaling via Kubernetes HPA.

4. **Databases**
   - **MongoDB**: Primary persistent data store for Users and Tasks.
   - **Redis**: In-memory message broker enabling decoupled, asynchronous communication between the Node.js API and Python workers.

## Scalability & Performance Strategy
To support 100,000 tasks/day (~70 tasks/minute on average, with potential spikes):

- **Horizontal Pod Autoscaling (HPA)**: Both the Backend API and Python Worker deployments are configured with HPA in Kubernetes, scaling automatically based on CPU utilization (target 70%).
- **Database Indexing**: MongoDB collections are optimized. The `Task` schema utilizes an index on `user` and `status` to rapidly fetch user-specific dashboards and filter pending tasks.
- **Asynchronous Decoupling**: Offloading processing to Python workers ensures the main Node event loop is never blocked, maintaining high throughput for incoming API requests.

## Deployment Strategy (GitOps)
- **CI/CD Pipeline**: GitHub Actions automatically builds and pushes multi-architecture Docker images to Docker Hub upon merges to the `main` branch.
- **Argo CD**: Continuously monitors the `ai-task-infra` repository. Any changes to the Kubernetes manifests are automatically synchronized and applied to the cluster, ensuring the live state matches the declarative source of truth.
