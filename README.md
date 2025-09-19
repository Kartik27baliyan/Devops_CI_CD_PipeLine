# End-to-End CI/CD Pipeline for a Containerized Python Application

A complete **Continuous Integration and Continuous Delivery (CI/CD)** pipeline implemented with Jenkins. This project automates the entire software delivery process: from code commit to building a tested, containerized artifact ready for deployment.

## 🎯 Project Objective

The objective of this repository is to demonstrate a real-world, automated CI/CD workflow that:
1.  **Automatically builds** and tests the application upon every code change.
2.  **Containerizes** the application using Docker, ensuring consistency across all environments (from development to production).
3.  **Produces a deployable artifact** (a Docker image) as the final output of the pipeline, which can be reliably deployed to any container runtime like Kubernetes, AWS ECS, or a simple server.

## 📁 Repository Structure
Devops_CI_CD_PipeLine/
├── app.py # Source code of the Python Flask application
├── requirements.txt # Python dependencies (Flask)
├── Dockerfile # Defines the container image for the application
├── Jenkinsfile # Pipeline-as-Code defining the entire CI/CD process
└── README.md # Project documentation (this file)

## ⚙️ Pipeline Architecture & Stages (The Full CI/CD Flow)

The entire process is defined as code in the `Jenkinsfile` and consists of the following sequential stages:

### 1. Clone Repository
- **Purpose:** Fetches the latest version of the source code from the version control system (GitHub).
- **Tools:** Jenkins `checkout scm` command.

### 2. Build and Test Application
This phase validates that the application code works before we bother containerizing it.
- **A. Python Environment Setup:**
    - Creates an isolated virtual environment and installs dependencies.
    - **Tools:** Python `venv`, `pip`.
- **B. Integration Testing:**
    - Starts the Flask server and runs a live test against the `/` endpoint to verify it returns the correct response (`Hello, DevOps World!...`).
    - **Tools:** `curl`, shell process management.
    - **Why it's important:** This "shift-left" testing approach catches bugs early, saving time and resources.

### 3. Build Docker Image
- **Purpose:** Uses the `Dockerfile` to create a portable, standardized container image containing the application and its exact runtime environment.
- **Tools:** Docker Engine, Docker CLI.
- **Process:** The `Dockerfile` performs the following steps:
    - Starts from an official `python:3.9-slim-buster` base image.
    - Copies the application code into the container.
    - Installs dependencies inside the container.
    - Exposes port 5000 and sets the command to start the Flask app.

### 4. Push Docker Image (To a Registry)
- **Purpose:** Stores the validated, versioned image in a central repository so it can be deployed to any environment.
- **Tools:** Docker CLI, configured credentials for a container registry (e.g., Docker Hub, Amazon ECR, Azure Container Registry).
- **Outcome:** The image is now accessible with a unique tag (e.g., `your-dockerhub-username/my-app:${BUILD_NUMBER}`).

## 🛠️ Technology Stack

| Component          | Technology Used                         | Purpose                                           |
| ------------------ | --------------------------------------- | ------------------------------------------------- |
| **Version Control** | GitHub                                  | Hosting source code and triggering webhooks.      |
| **CI/CD Server**    | Jenkins                                 | Orchestrating the entire pipeline execution.      |
| **Pipeline Definition** | Jenkinsfile (Groovy)              | Defining the pipeline stages as code (IaC).       |
| **Application**     | Python 3, Flask                         | The web application being built and tested.       |
| **Containerization**| Docker                                  | Creating a portable, dependency-free artifact.    |
| **Registry**        | Docker Hub / ECR / ACR                  | Storing and versioning container images.          |
| **Testing**         | Shell Scripting, `curl`                 | Application health and integration testing.       |

## 🚀 How the Pipeline Works (Flowchart)

```mermaid
graph LR
A[Git Push to GitHub] --> B{Jenkins Pipeline Triggered};
B -- Clone Repo --> C[Stage 1: Clone Code];
C --> D[Stage 2: Build & Test];
D -- Tests Pass --> E[Stage 3: Build Docker Image];
E --> F[Stage 4: Push Image to Registry];
F --> G[✅ Pipeline Success];
D -- Tests Fail --> H[❌ Pipeline Failed];
H --> I[Developer Gets Alert];
