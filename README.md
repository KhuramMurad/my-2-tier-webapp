```markdown
# 2-Tier Node.js Application

This project is a 2-tier web application consisting of a Node.js frontend, a Node.js backend, and a PostgreSQL database, all deployed on a local Kubernetes cluster using Minikube and managed with a single Helm chart. The application demonstrates a simple full-stack setup with containerized components.

- **Frontend**: A Node.js app serving a basic webpage, communicating with the backend.
- **Backend**: A Node.js API that connects to the PostgreSQL database.
- **Database**: PostgreSQL for persistent data storage (persistence disabled by default for simplicity).

This README provides step-by-step instructions to set up, deploy, and access the application on your local machine.

---

## Prerequisites

Before starting, ensure you have the following tools installed on your system:

- **[Minikube](https://minikube.sigs.k8s.io/docs/start/)**: A tool to run a single-node Kubernetes cluster locally.
- **[Helm](https://helm.sh/docs/intro/install/)**: A package manager for Kubernetes to deploy the app.
- **[Docker](https://docs.docker.com/get-docker/)**: Used by Minikube to build and run containers.
- **[kubectl](https://kubernetes.io/docs/tasks/tools/)**: The Kubernetes command-line tool to interact with the cluster.
- **Git**: To clone this repository.

### Installation Commands (Ubuntu Example)
```bash
# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

# Install Docker
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install kubectl /usr/local/bin/kubectl

# Install Git
sudo apt install -y git
```

Verify installations:
```bash
minikube version
helm version
docker --version
kubectl version --client
git --version
```

---

## Setup

Follow these steps to clone the repository and set up the environment.

### 1. Clone the Repository
Clone this project to your local machine:
```bash
git clone git@github.com:KhuramMurad/my-2-tier-webapp.git
cd my-2-tier-webapp
```

### 2. Start Minikube
Launch a local Kubernetes cluster using Minikube with the Docker driver:
```bash
minikube start --driver=docker
```
- **Explanation**: This command starts a single-node Kubernetes cluster inside a Docker container. The `--driver=docker` flag uses Docker as the virtualization layer, suitable for most Linux systems.

Set up the Docker environment for Minikube:
```bash
eval $(minikube -p minikube docker-env)
```
- **Explanation**: This configures your terminal to use Minikube’s Docker daemon, ensuring containers are built within the Minikube environment.

Verify Minikube is running:
```bash
minikube status
```
- Expect: `host: Running`, `kubelet: Running`, `apiserver: Running`.

---

## Deploy the Application

Deploy the 2-tier application using Helm.

### 1. Install the Helm Chart
Run the following command from the project root:
```bash
helm install two-tier-app ./helm
```
- **Explanation**: 
  - `helm install`: Deploys the Helm chart.
  - `two-tier-app`: The release name for this deployment.
  - `./helm`: The directory containing the Helm chart (`Chart.yaml`, `values.yaml`, and `templates/`).

### 2. Verify Deployment
Check the status of the deployed pods and services:
```bash
kubectl get pods
```
- Expect output like:
  ```
  NAME                            READY   STATUS    RESTARTS   AGE
  two-tier-app-db-...             1/1     Running   0          2m
  two-tier-app-backend-...        1/1     Running   0          2m
  two-tier-app-frontend-...       1/1     Running   0          2m
  ```

Check the services:
```bash
kubectl get svc
```
- Expect output like:
  ```
  NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
  two-tier-app-db         ClusterIP   10.x.x.x        <none>        5432/TCP         2m
  two-tier-app-backend    ClusterIP   10.x.x.x        <none>        4000/TCP         2m
  two-tier-app-frontend   NodePort    10.x.x.x        <none>        3000:30001/TCP   2m
  ```

---

## Access the Application

You can access the frontend in your browser using one of two methods. The recommended method is simpler and more reliable.

### Recommended: Minikube Service
Expose the frontend service to your local machine:
```bash
minikube service two-tier-app-frontend
```
- **Explanation**: This command creates a tunnel from your host to the Kubernetes service, automatically opening the app in your default browser (e.g., at `http://127.0.0.1:<port>` where `<port>` is dynamically assigned, like `46785`).
- **Outcome**: You’ll see a message like "Hello from Frontend! Backend is at http://backend-backend:4000".

To get the URL manually:
```bash
minikube service two-tier-app-frontend --url
```
- Copy the output (e.g., `http://127.0.0.1:46785`) and open it in your browser.

### Alternative: NodePort
If you prefer direct access via Minikube’s IP:
1. Get the Minikube IP:
   ```bash
   minikube ip
   ```
   - Example output: `192.168.49.2`.

2. Access the app:
   - Open `http://<minikube-ip>:30001` in your browser (e.g., `http://192.168.49.2:30001`).
   - **Note**: This may require additional networking setup (see below).

3. Ensure port `30001` is open:
   ```bash
   sudo iptables -A INPUT -p tcp --dport 30001 -j ACCEPT
   sudo iptables -A OUTPUT -p tcp --dport 30001 -j ACCEPT
   sudo iptables-save > /etc/iptables/rules.v4
   sudo ufw allow 30001
   ```
   - **Explanation**: These commands configure your firewall to allow traffic to port `30001`. Persistence varies by system; adjust as needed.

- **Caveat**: `NodePort` access might not work consistently due to local networking configurations (e.g., Docker bridge isolation). Use `minikube service` if issues persist.

---

## Project Structure

- **`helm/`**: Contains the Helm chart.
  - `Chart.yaml`: Metadata for the Helm chart.
  - `values.yaml`: Configuration values (e.g., image names, ports).
  - `templates/`**: Kubernetes manifests for the database, backend, and frontend.
- **`README.md`**: This file.

---

## Notes

- **Container Images**:
  - Frontend: `khurammurad/frontend:latest`
  - Backend: `khurammurad/backend:latest`
  - Database: `postgres:15`
- **Persistence**: Disabled in `values.yaml` (`persistence.enabled: false`) to avoid PVC issues in Minikube. To enable:
  - Set `persistence.enabled: true` in `values.yaml`.
  - Ensure Minikube’s storage provisioner works (e.g., `kubectl get sc` shows `standard`).
- **Access Preference**: Use `minikube service` for reliable browser access on local setups.

---

## Cleanup

To remove the application and stop the cluster:
```bash
helm uninstall two-tier-app
minikube stop
```
- **Explanation**:
  - `helm uninstall`: Deletes the Helm release and associated Kubernetes resources.
  - `minikube stop`: Shuts down the Minikube cluster.

To delete the Minikube cluster entirely:
```bash
minikube delete
```

---

## Troubleshooting

- **Pods Not Running**:
  - Check logs: `kubectl logs <pod-name>`.
  - Describe pod: `kubectl describe po <pod-name>`.

- **Browser Timeout at NodePort**:
  - Verify connectivity: `curl -v http://192.168.49.2:30001`.
  - Use `minikube service` instead.

- **Minikube Issues**:
  - Restart: `minikube stop && minikube start --driver=docker`.

For further assistance, open an issue on this repository.

---

## Contributing

Feel free to fork this repository, make improvements, and submit pull requests. Contributions are welcome!

---
```

---

### Step 1: Update Your Local README
1. **Replace Your Current README**:
   ```bash
   cd ~/my-2tier-nodejs-app
   cat <<EOF > README.md
   # Paste the entire content from above here
   EOF
   ```
   - Copy the markdown content from above and paste it into the `EOF` block, then run the command. Alternatively, edit `README.md` manually in a text editor like `nano` or `vim`.

2. **Commit and Push**:
   ```bash
   git add README.md
   git commit -m "Add detailed README with setup and access instructions"
   git push
   ```

---

### Step 2: Verify on GitHub
- Visit `https://github.com/KhuramMurad/my-2-tier-webapp`.
- Confirm the updated `README.md` renders correctly with all sections (use the preview if needed).

---

### Step 3: Test the Instructions
To ensure the README works as expected:
```bash
helm uninstall two-tier-app
minikube stop
minikube start --driver=docker
eval $(minikube -p minikube docker-env)
helm install two-tier-app ./helm
minikube service two-tier-app-frontend
```
- Verify the app opens in your browser.

---

### Confirmation
This `README.md` provides a comprehensive guide for anyone to set up and run your project. It emphasizes `minikube service` for access, as that’s what works reliably for you, and includes `NodePort` as an optional method with caveats. Let me know:
- If you’ve pushed this to GitHub successfully.
- If the app still runs fine with these instructions.
- Any adjustments you’d like to make.

Your project is now well-documented and ready for others to use! Share the outcome or any tweaks you need.
