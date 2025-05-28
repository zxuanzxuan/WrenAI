# Deploying WrenAI on Google Cloud Engine (GCE) with Vertex AI

This guide explains how to deploy WrenAI using Docker Compose on a GCE VM, leveraging Google Cloud Vertex AI for Large Language Models and Embeddings.

## Prerequisites

1.  **Google Cloud Project:** You need an active GCP project.
2.  **GCE VM:**
    *   Create a new GCE VM instance in your desired region (e.g., `us-central1`).
    *   Ensure the VM has sufficient resources (e.g., e2-standard-4 or higher is recommended).
    *   **Service Account Permissions:** The service account associated with your GCE VM must have the necessary IAM permissions to use Vertex AI. Grant the "Vertex AI User" role to the VM's service account. This allows it to make predictions and use embedding models.
3.  **Install Docker and Docker Compose:**
    *   SSH into your GCE VM.
    *   Follow the official Docker documentation to install Docker Engine: [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/) (select the appropriate Linux distribution).
    *   Install Docker Compose: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)
4.  **Clone this Repository:** Clone this repository (or your fork) onto your GCE VM:
    ```bash
    git clone <your-repository-url>
    cd <repository-name>/docker
    ```

## Configuration

1.  **Docker Compose File:**
    *   This deployment uses `docker-compose.gcp.yaml`, which is pre-configured to run `wren-engine`, `wren-ai-service`, `wren-ui`, and `qdrant`.

2.  **AI Service Configuration:**
    *   The AI service is configured in `config.gcp.yaml`.
    *   It's set up to use Google Vertex AI models:
        *   **LLM:** `gemini-2.5-pro-preview-05-06` (via `vertex_ai/gemini-2.5-pro-preview-05-06`)
        *   **Embedding Model:** `textembedding-gecko@003` (via `vertex_ai/textembedding-gecko@003`)
    *   The GCP `project_id` (`james-sandbox-317709`) and `location` (`us-central1`) are specified in this file.

3.  **Environment Variables (`docker/.env.gcp`):**
    *   This file contains environment variables for the Docker Compose setup.
    *   **Crucial Vertex AI Variables:**
        *   `VERTEX_AI_PROJECT_ID="james-sandbox-317709"`
        *   `VERTEX_AI_LOCATION="us-central1"`
        These should match your GCP setup.
    *   **API Keys for Other Services:** If you plan to use other services that require API keys (e.g., OpenAI, Langfuse), you must fill in the placeholder values (e.g., `OPENAI_API_KEY`, `LANGFUSE_SECRET_KEY`, `LANGFUSE_PUBLIC_KEY`). For Vertex AI, authentication is primarily handled by the GCE VM's service account.
    *   **Service Versions:** Review and update the service versions (e.g., `WREN_AI_SERVICE_VERSION`, `WREN_UI_VERSION`) to the desired releases.
    *   **Ports:** Default ports are `HOST_PORT=3000` (for Wren UI) and `AI_SERVICE_FORWARD_PORT=5555` (for Wren AI Service). Adjust if needed due to port conflicts on your VM.

## Deployment Steps

1.  **Navigate to the Docker Directory:**
    If you are not already there, `cd` into the `docker` directory where `docker-compose.gcp.yaml` is located.
    ```bash
    cd /path/to/your-repo/docker
    ```

2.  **Start the Services:**
    Run the following command to build (if necessary) and start all services in detached mode:
    ```bash
    docker compose -f docker-compose.gcp.yaml --env-file .env.gcp up -d
    ```
    *   The `-f` flag specifies the custom Docker Compose file.
    *   `--env-file` specifies the custom environment file.
    *   `-d` runs the containers in the background.

3.  **Check Container Status:**
    To see the status of your running containers:
    ```bash
    docker compose -f docker-compose.gcp.yaml --env-file .env.gcp ps
    ```
    To view logs for all services:
    ```bash
    docker compose -f docker-compose.gcp.yaml --env-file .env.gcp logs -f
    ```
    To view logs for a specific service (e.g., `wren-ai-service`):
    ```bash
    docker compose -f docker-compose.gcp.yaml --env-file .env.gcp logs -f wren-ai-service
    ```

## Accessing WrenAI

*   **Wren UI:** Open your web browser and navigate to `http://<your-gce-vm-external-ip>:<HOST_PORT>` (e.g., `http://YOUR_VM_IP:3000`).
*   **Wren AI Service API:** The AI service will be available at `http://<your-gce-vm-external-ip>:<AI_SERVICE_FORWARD_PORT>` (e.g., `http://YOUR_VM_IP:5555`).

Ensure your GCE VM's firewall rules allow inbound TCP traffic on the ports you've configured (e.g., 3000 and 5555).

## Stopping and Cleaning Up

*   **Stop Services:**
    ```bash
    docker compose -f docker-compose.gcp.yaml --env-file .env.gcp down
    ```
*   **Remove Volumes (optional, deletes all data):**
    If you also want to remove the data volumes (e.g., Qdrant database, Wren UI SQLite DB):
    ```bash
    docker compose -f docker-compose.gcp.yaml --env-file .env.gcp down -v
    ```

## Notes

*   The configuration in `config.gcp.yaml` uses LiteLLM to interface with Vertex AI. Ensure that the model names and parameters are compatible with LiteLLM's Vertex AI integration.
*   If you encounter issues, checking the logs of individual services is the first step in troubleshooting.
