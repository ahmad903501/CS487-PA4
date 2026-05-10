<div align="center">

# PA4 Submission: TaskFlow Pipeline

<img alt="GitHub only" src="https://img.shields.io/badge/Submit-GitHub%20URL%20Only-10b981?style=for-the-badge">
<img alt="Total points" src="https://img.shields.io/badge/Total-100%20points-7c3aed?style=for-the-badge">

</div>

<div style="background:#f5f3ff;color:#111827;border-left:6px solid #6330bc;padding:14px 18px;border-radius:10px;margin:18px 0;">
Copy this file to <code style="color:#111827;background:#ddd6fe;padding:2px 4px;border-radius:4px;">SUBMISSION.md</code>. Put every screenshot in <code style="color:#111827;background:#ddd6fe;padding:2px 4px;border-radius:4px;">docs/</code>, embed it under the correct task, and write a short description below each image explaining what it proves. The grader should not need any file outside this repository.
</div>

## Student Information

| Field | Value |
|---|---|
| Name | Ahmad Jawwad |
| Roll Number | 25100237 |
| GitHub Repository URL | https://github.com/ahmad903501/CS487-PA4 |
| Resource Group | `rg-sp26-25100237` |
| Assigned Region | `ukwest` |

## Evidence Rules

- Use relative image paths, for example: `![AKS nodes](docs/aks-nodes.png)`.
- Every image must have a 1-3 sentence description below it.
- Azure Portal screenshots must show the resource name and enough page context to identify the service.
- CLI screenshots must show the command and output.
- Mask secrets such as function keys, ACR passwords, and storage connection strings.


## Task 1: App Service Web App (15 points)

### Evidence 1.1: Forked Repository

![Forked repository](docs/StarterCodeFork.png)

Description: This screenshot shows my GitHub fork (`ahmad903501/CS487-PA4`) containing the required starter folders (`webapp`, `function-app`, `validate-api`, `report-job`, and `docs`). It confirms I worked in my own fork as required by the assignment.

### Evidence 1.2: App Service Overview

![Web App overview](docs/ApplicationCentreConfigured.png)

Description: The Web App `pa4-25100237` is deployed in resource group `rg-sp26-25100237` and region `UK West`, with status Running. The public endpoint is hosted on `azurewebsites.net` and serves the TaskFlow UI over HTTPS.

### Evidence 1.3: Deployment Center / GitHub Actions

![Deployment Center GitHub config](docs/DeploymentCentreConfiguration.png)

Description: Deployment Center is configured to pull from my GitHub repository `ahmad903501/CS487-PA4` on the `main` branch using GitHub Actions. This proves CI/CD wiring from GitHub to App Service.

### Evidence 1.4: Live Web UI

![Live TaskFlow UI](docs/WebAppRunning.png)

Description: The TaskFlow form loads successfully in the browser, confirming the App Service is serving the frontend correctly. At early stage it showed expected configuration errors before backend wiring.

---

## Task 2: Azure Container Registry (15 points)

### Evidence 2.1: ACR Overview

![ACR overview](docs/ACR_Succeeded.png)

Description: The ACR `pa425100237` is provisioned successfully in resource group `rg-sp26-25100237` with SKU `Basic`.

### Evidence 2.2: Docker Builds

![Docker builds completed](docs/Successfulydockerbuilds.png)

Description: The images were built from these folders: `validate-api` from `validate-api/`, `report-job` from `report-job/`, and `func-app` from `function-app/`. The screenshot confirms all three builds completed.

### Evidence 2.3: ACR Repositories

![ACR repositories list](docs/ACR_List.png)

Description: The repository list shows all required repos in ACR: `validate-api`, `report-job`, and `func-app`. These correspond to pushed image tags `validate-api:v1`, `report-job:v1`, and `func-app:v1`.

---

## Task 3: Durable Function Implementation (12 points)

### Evidence 3.1: Completed Function Code

[function_app.py](function-app/function_app.py)

Description: The orchestrator receives the order input, calls `validate_activity`, branches on `validation.valid`, and either returns a rejected response or calls `report_activity`. The report activity creates an ACI container group, waits for completion, deletes the container group, and returns the final blob URL.

### Evidence 3.2: Local Function Handler Listing

![Local func start handlers](docs/functionsregistered.png)

Description: `func start` lists `http_starter`, `my_orchestrator`, `validate_activity`, and `report_activity`. This confirms the Durable runtime discovered all required triggers and activities.

---

## Task 4: Function App Container Deployment (8 points)

### Evidence 4.1: Function App Container Configuration

![Function App container config](docs/functionappcontainerimageconfiguration.png)

Description: Function App `pa4-2500237-func` is configured to use image `pa425100237.azurecr.io/func-app:v1` from Azure Container Registry with admin credentials.

### Evidence 4.2: Orchestration Smoke Test

![Orchestration smoke test curl](docs/function_curloutput.png)

Description: The returned `id` is the orchestration instance ID and `statusQueryGetUri` is the polling endpoint for runtime status. This proves the HTTP starter successfully created a Durable orchestration instance.

### Evidence 4.3: Expected Failed Status Before Downstream Wiring

![Expected failed status](docs/statusqueryurljson.png)

Description: The failed state with `KeyError: 'VALIDATE_URL'` is expected when downstream app settings are not fully configured. It confirms function execution is working but environment configuration was incomplete at that step.

---

## Task 5: AKS Validator (15 points)

### Evidence 5.1: AKS Cluster

![AKS cluster evidence](docs/rg_alldeployments.png)

Description: The AKS resource `pa4-25100237` is deployed in `rg-sp26-25100237`, region `UK West`. Node evidence (`kubectl get nodes`) shows one ready node and cluster version output in the terminal.

### Evidence 5.2: Kubernetes Nodes and Pods

![Kubernetes nodes and pods](docs/kubectlgetpod.png)

![Kubernetes nodes](docs/kubectgetnodess.png)

Description: `kubectl get nodes` shows a Ready node and `kubectl get pods` shows the validator deployment pod in Running state. This confirms scheduling and healthy pod startup on AKS.

### Evidence 5.3: Kubernetes Service

![Kubernetes service external IP](docs/kubectlgetserviceandhealthandvalidateruns.png)

Description: The `validate-service` LoadBalancer exposes port `8080` with an external IP used by the Function App for validation calls.

### Evidence 5.4: Validator API Tests

![Validator API curl tests](docs/localtestvalidator.png)

Description: `/health` returns `{"status":"ok"}`. A valid order with `qty <= 100` returns `{"valid": true}`, and an invalid order with `qty > 100` returns `{"valid": false, "reason": "quantity exceeds limit"}`.

### Evidence 5.5: Function App `VALIDATE_URL`

![Function app VALIDATE_URL](docs/functionappvalidateurl.png)

Description: `VALIDATE_URL` is configured in Function App environment variables to point to the AKS validator endpoint. `validate_activity` reads this setting and POSTs the incoming order payload to `/validate`.

### Evidence 5.6: AKS Idle Behavior

![AKS idle behavior](docs/kubectgetnodess.png)

Description: Even when no requests are being processed, `kubectl get nodes` still shows the node in Ready state. This demonstrates that AKS worker nodes stay provisioned and bill continuously while the cluster exists.

---

## Task 6: ACI Report Job (15 points)

### Evidence 6.1: Blob Container

![Blob reports container](docs/reportscontainercreated.png)

Description: The `reports` blob container in storage account `pa425100237` is the target location where report-job uploads generated PDFs.

### Evidence 6.2: Manual ACI Run

![Manual ACI run status](docs/azcontainrelogs_reportgenerated.png)

Description: `az container show --query instanceView.state` returns `Succeeded` for `ci-report-test`. This is expected because the report-job is a one-shot batch container that exits after generating and uploading the report.

### Evidence 6.3: ACI Logs

![ACI logs upload confirmation](docs/containerlogsreportsjob.png)

Description: The logs include `Uploaded TEST-001.pdf to reports container`, confirming the container completed report generation and blob upload successfully.

### Evidence 6.4: Generated PDF

![Generated PDF evidence](docs/pdfreportgenerated.png)

Description: The `TEST-001.pdf` artifact is visible/openable from Blob Storage. This proves the ACI job wrote a real output file to the `reports` container.

### Evidence 6.5: Function App Managed Identity and IAM

![Function app managed identity](docs/userassignedidentity_funcapp.png)

![Resource group IAM role assignment](docs/rg_alldeployments.png)

Description: The Function App uses managed identity to authenticate to Azure Resource Manager without hard-coded credentials. Contributor permission at resource-group scope is required so `report_activity` can create and delete ACI container groups dynamically.

### Evidence 6.6: Report App Settings

![Function app report settings](docs/functionappapplicationsettings.png)

Description: `REPORT_*` defines target RG/region/image for ACI creation, `ACR_*` enables pulling private images, `STORAGE_ACCOUNT_URL` points to Blob Storage for report output, and `SUBSCRIPTION_ID` identifies the Azure subscription for ARM operations. Secrets are masked in screenshots.

---

## Task 7: End-to-End Pipeline (15 points)

### Evidence 7.1: Web App Wiring

![Web app function URL settings](docs/ApplicationCentreConfigured.png)

Description: The Web App app settings contain `FUNCTION_START_URL` and `FUNCTION_STATUS_URL`. The frontend submits orders to the start URL, then polls orchestration status using the status URL prefix.

### Evidence 7.2: Happy Path UI

![Happy path before submit](docs/WebAppRunning.png)

![Happy path completed](docs/pdfreport.png)

Description: A valid payload such as `{"order_id":"ORD-003","items":[{"sku":"WIDGET-Y","qty":2}]}` completes successfully. The UI shows Completed state and a report link, confirming orchestration reached ACI and blob output.

### Evidence 7.3: Backend Participation

![Function app invocation/log evidence](docs/detailedlogs_7.2step.png)

![AKS validator pod logs](docs/detaillogsakspod.png)

![ACI execution logs](docs/azcontainrelogs_reportgenerated.png)

![Blob report evidence](docs/pdfreportgenerated.png)

Description: The same order ID is visible across Function execution output, AKS validation calls, ACI logs, and Blob report artifacts. This confirms full participation of all backend services in one end-to-end flow.

### Evidence 7.4: Reject Path UI

![Reject path UI](docs/rejectedpath.png)

Description: Orders with `qty > 100` are rejected by `validate_activity` response. The orchestrator returns rejected status immediately and does not invoke `report_activity`, so no ACI is created for rejected requests.

---

## Task 8: Write-up and Architecture Diagram (5 points)

### Evidence 8.1: Architecture Diagram

![Architecture diagram](docs/architecturediagram.png)

Description: The architecture diagram shows GitHub-driven App Service deployment, Durable Function orchestration, AKS validator, ACI report job, Blob Storage for reports, ACR for container images, and identity/IAM flow for secure service access.

### Question 8.2: Service Selection

App Service is used for the web frontend because it is a managed PaaS with straightforward CI/CD from GitHub and stable HTTPS hosting for user-facing traffic. It is a good fit for a continuously available UI without managing VMs or cluster infrastructure.

Durable Functions is used as the orchestration layer because this workflow is stateful and multi-step (`validate` then `report`). Durable checkpoints orchestration progress between activities, supports replay safely, and provides built-in status endpoints for client polling.

AKS is used for the validator API because it is a long-lived HTTP microservice that benefits from Kubernetes deployment/service primitives and controlled scaling behavior. It is a strong fit for always-on internal APIs that may evolve into more advanced containerized services.

ACI is used for report generation because report creation is bursty and short-lived. Each request can launch an isolated container, do the work, write output to Blob, and exit, avoiding idle compute cost.

### Question 8.3: ACI vs AKS

AKS remains provisioned when idle: nodes stay running and continue incurring VM-related costs even when no orders are processed. Operationally, AKS is cluster-based and suited for continuously available services.

ACI is run-to-completion in this design: a container group is created for a job and deleted after completion. When no jobs run, no container instance exists, so compute cost is effectively zero for that component.

From an operations perspective, AKS offers more control and platform features, while ACI provides simpler ephemeral execution with less infrastructure management. In this assignment, AKS is appropriate for validator uptime and ACI is appropriate for one-shot report jobs.

### Question 8.4: Durable Functions vs Plain HTTP

Durable Functions solves state persistence for multi-step workflows, so the system does not need custom storage logic to remember whether an order has passed validation before launching report generation.

Durable Functions also addresses long-running orchestration concerns: the HTTP starter returns immediately with status URLs while the backend workflow continues asynchronously. This avoids fragile client-held waits and reduces timeout-related issues typical of plain chained HTTP handlers.

Additionally, Durable replay and checkpointing make the flow more resilient if an activity temporarily fails, which would otherwise require custom retry and compensation logic.

### Question 8.5: Cost Review

![Cost analysis](docs/rg_alldeployments.png)

Description: The most expensive component is AKS because worker nodes remain allocated continuously. Estimated PA consumption during development and testing was approximately USD 5-8, with AKS and related networking dominating cost.

### Question 8.6: Challenges Faced

1. While using MINGW64 on Windows, path conversion corrupted Azure resource IDs and caused `LinkedInvalidPropertyId` style failures. I debugged by inspecting command payload behavior and fixing it with `MSYS_NO_PATHCONV=1` for affected commands.

2. I hit managed identity permission propagation delay after role assignment, which caused authorization failures when creating ACI from the Function App. I validated identity wiring in Function logs, waited for RBAC propagation, and restarted the Function App to clear stale auth context.

3. During local API testing in PowerShell, `curl` was aliased to `Invoke-WebRequest`, causing header/body parsing errors. I switched to `curl.exe` with properly escaped JSON for reliable request behavior.

---
