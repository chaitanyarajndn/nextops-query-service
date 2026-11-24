# NextOps Industrial - Telemetry Query Service

## Introduction

The NextOps Query Service is the backend API for the NextOps Industrial monitoring dashboard. It reads equipment telemetry from a PostgreSQL database and provides that data to the frontend through FastAPI endpoints.

The data flow works as follows:

1. Industrial equipment sends or stores telemetry in the PostgreSQL database.
2. The Query Service connects to PostgreSQL through `config.py` and runs queries from `database.py`.
3. The FastAPI application in `main.py` exposes the results as HTTP API endpoints.
4. The NextOps React frontend calls these endpoints to display dashboards, devices, trends, regions, and alarms.

For example, the frontend requests `/dashboard/summary`, and the Query Service returns the latest telemetry summary calculated from the database. The service normally runs on port `8000`, allowing the frontend to access it over the VM public IP.

## Simple Setup on Azure VM (Without Docker)

This guide explains how to set up and run the FastAPI service directly on an Azure virtual machine.

---

## 1. Open Port 8000 in Azure NSG

In the Azure Portal:
1. Open your Virtual Machine, then go to **Networking** > **Network settings** (or open the associated **Network Security Group**).
2. In **Inbound port rules**, select **Add inbound rule** and enter:
   - **Destination port ranges**: `8000`
   - **Protocol**: `TCP`
   - **Action**: `Allow`
   - **Priority**: `1010`
   - **Name**: `Allow-Port-8000`
3. Confirm that **Port 22 (SSH)** is also allowed so you can connect to the VM.

---

## 2. Connect to the Azure VM

```bash
ssh azureuser@<YOUR_AZURE_PUBLIC_IP>
```

---

## 3. Install Python, Pip & Git

```bash
sudo apt update
sudo apt install -y python3 python3-pip git
```

---

## 4. Clone the Repository

```bash
git clone <YOUR_REPO_URL>
cd nextops-query-service
```

---

## 5. Update Connection String in `config.py`

Open `config.py` to add the PostgreSQL connection details:

```bash
nano config.py
```

Replace the sample database URL with your existing PostgreSQL connection string:

```python
    DATABASE_URL: str = os.getenv(
        "DATABASE_URL",
        "postgresql://<username>:<password>@<db-host>:<port>/<database-name>",
    )
```

*(Note: If your password contains special characters like `@`, URL-encode them, e.g. `@` becomes `%40`)*

Save your changes and exit Nano (`Ctrl+O`, `Enter`, `Ctrl+X`).

---

## 6. Install Dependencies

Install the Python packages listed in `requirements.txt`:

```bash
pip install -r requirements.txt
```

*(Note: On Ubuntu 24.04 or newer, Python may show an externally-managed-environment notice. If it appears, add `--break-system-packages` to the command: `pip install -r requirements.txt --break-system-packages`)*

---

## 7. Install Uvicorn

```bash
pip install uvicorn
```

---

## 8. Run the Application with Uvicorn

### Option A: Run in the Foreground for Testing

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

### Option B: Run in the Background (Keeps running after you disconnect SSH)

```bash
nohup uvicorn main:app --host 0.0.0.0 --port 8000 > app.log 2>&1 &
```

- **Check logs:**
  ```bash
  tail -f app.log
  ```
- **Stop the app:**
  ```bash
  pkill -f uvicorn
  ```

---

## 9. Test in Browser

Open the following URLs in a browser. Replace `<YOUR_AZURE_PUBLIC_IP>` with the VM's public IP address:

- **Swagger API Docs:** `http://<YOUR_AZURE_PUBLIC_IP>:8000/docs`
- **Healthcheck:** `http://<YOUR_AZURE_PUBLIC_IP>:8000/health`
- **Dashboard Summary:** `http://<YOUR_AZURE_PUBLIC_IP>:8000/dashboard/summary`
