# Airflow GCP — ML Pipeline Project

## Project Overview

This project is an **Apache Airflow** pipeline that automates an end-to-end **machine learning workflow** for an advertising click-prediction use case. The DAG orchestrates the following steps:

1. **Load Data** — Reads advertising data from a CSV file (user ad-click behavior).
2. **Preprocess** — Splits data into train/test sets and applies feature scaling (MinMaxScaler, StandardScaler).
3. **Build Model** — Trains an SVM classifier and saves it as `model.sav`.
4. **Evaluate** — Computes accuracy, classification report, and confusion matrix on the test set.
5. **Predict** — Runs predictions on the test data.
6. **Notify** — Sends a success email when the pipeline completes.

The DAG (`Airflow_Lab3_new2`) chains these tasks together and passes data between them using XCom. Supporting scripts include `model_development.py` (data loading, preprocessing, model training, evaluation) and `success_email.py` (SMTP email notifications).

---

## Step-by-Step Guide: Deploying and Triggering Airflow DAGs (Standalone Mode)

This guide covers setting up Apache Airflow in **standalone mode** on a virtual machine. Standalone mode runs the webserver, scheduler, and triggerer in a single process—ideal for development and small deployments.

### 1. Create and Configure a VM

1. **Create a Virtual Machine Instance**:
   - Log in to your cloud provider and create a new VM instance with sufficient resources (e.g., 2 vCPUs, 4GB RAM) to handle Airflow.

2. **Set Up Networking**:
   - Go to **VPC Network** and add a firewall rule to allow HTTP (port 80) and custom Airflow webserver port (e.g., port 8080).

---

### 2. Update and Install Necessary Packages

After the VM is set up, SSH into it and update the system, then install necessary packages:

```bash
sudo apt update
sudo apt install python3-pip python3-venv python3-full -y
```

---

### 3. Set Up a Virtual Environment for Airflow

1. **Create a Virtual Environment**:
   ```bash
   python3 -m venv airflow_new_venv
   ```

2. **Activate the Virtual Environment**:
   ```bash
   source airflow_new_venv/bin/activate
   ```

3. **Install Apache Airflow**:
   ```bash
   pip install apache-airflow
   ```
4. Initialize the Airflow Database:
   airflow db migrate
---

### 4. Start Airflow in Standalone Mode

**Airflow standalone** runs the webserver, scheduler, and triggerer in a single process. On first run, it automatically initializes the database and creates an admin user.

1. **Activate the Virtual Environment** (if not already active):
   ```bash
   source airflow_new_venv/bin/activate
   ```

2. **Start Airflow Standalone**:
   ```bash
   airflow standalone
   ```

On first run, Airflow will:
- Initialize the database
- Create an admin user
- Print the **username** and **password** in the terminal—save these for logging in
- If not printed there it will be in a file named simple_auth_manager_passwords. use the below command to get it.
- cat ~/airflow/simple_auth_manager_passwords.json.generated

The Airflow web interface will be accessible at `http://<VM-IP>:8080`. Log in with the credentials printed in the terminal.

---

### 5. Enable the Airflow API (Optional)

To enable the Airflow API for programmatic access, configure the `airflow.cfg` file:

1. Open the `airflow.cfg` file for editing:
   ```bash
   nano ~/airflow/airflow.cfg
   ```

2. Locate the `[api]` section and modify it to enable the API:
   ```ini
   [api]
   auth_backend = airflow.api.auth.backend.basic_auth
   ```

---

### 6. Set Up Folder Structure for Airflow Project

1. In your Airflow directory, create folders for DAGs and requirements:
   ```bash
   mkdir dags
   mkdir dags/src
   touch requirements.txt
   ```

2. **Add Python Scripts for DAGs**:
   - Place your DAG Python script, `my_dag.py`, in the `dags` folder.
   - Place other required scripts, like `model_development.py` and `success_email.py`, in the `dags/src` folder.
   - Ensure the `advertising.csv` data file is in `dags/data/`.

   Example structure:
   ```plaintext
   airflow/
   ├── dags/
   │   ├── my_dag.py
   │   ├── data/
   │   │   └── advertising.csv
   │   └── src/
   │       ├── model_development.py
   │       └── success_email.py
   └── requirements.txt
   ```

---

### 7. Configure Email (SMTP) for Notifications

The DAG sends a success email when the pipeline completes. The `success_email.py` task uses an Airflow connection for credentials. Configure email **before** triggering the DAG.

#### Sign in with App Passwords (Gmail)

For Gmail, you must use an [App Password](https://support.google.com/accounts/answer/185833) if you have 2-Step Verification enabled—your regular password will not work. Follow the instructions at the link to generate your SMTP password.

#### Add Email Connection
Instead of putting your username/password in airflow.cfg, use the connection command to store them securely:

Add the `email_smtp` connection with your Gmail address and App Password:

```bash
airflow connections add 'smtp_default' \
    --conn-type 'email' \
    --conn-host 'smtp.gmail.com' \
    --conn-port '587' \
    --conn-login 'YOUREMAIL@gmail.com' \
    --conn-password 'YOUR_APP_PASSWORD'
```

Replace `YOUREMAIL@gmail.com` with your Gmail address and `your-app-password` with the App Password you generated.

#### (Optional) Add SMTP Information to airflow.cfg

For Airflow's built-in email features, you can also configure the `[smtp]` section in `airflow.cfg`:

1. Open the file:
   ```bash
   nano ~/airflow/airflow.cfg
   ```
2. Search for the `[smtp]` section and update with your Gmail settings:

```ini
[smtp]
smtp_host = smtp.gmail.com
smtp_starttls = True
smtp_ssl = False
smtp_user = YOUREMAIL@gmail.com
smtp_password = Enter your password generated above
smtp_port = 587
smtp_mail_from = YOUREMAIL@gmail.com
smtp_timeout = 30
smtp_retry_limit = 5
```

---

### 8. Create and Edit DAG Files

1. **Define DAG**:
   Create and edit the `my_dag.py` DAG file:
   ```bash
   nano dags/my_dag.py
   ```

   Sample `my_dag.py` content:
   ```python
   Code added in files
   ```

2. **Additional Python Scripts**:
   Edit and add any additional scripts required, such as `model_development.py` and `success_email.py`.

---

### 9. Install Requirements

If you have any specific Python packages listed in `requirements.txt`, install them with:

```bash
pip install -r requirements.txt
```

---

### 10. Trigger the DAG

1. **Activate the Virtual Environment**:
   ```bash
   source airflow_new_venv/bin/activate
   ```

2. **Trigger the DAG Manually**:
   ```bash
   airflow dags trigger Airflow_Lab3_new2
   ```

You should see logs confirming that the DAG has been triggered. You can also monitor the DAG execution on the Airflow web interface.

---

### 11. Run and Test the DAG Locally

To test `model_development.py` directly (without Airflow):

```bash
python3 dags/src/model_development.py
```

This command will run the script locally and can help debug any issues before deploying it fully to Airflow.

---