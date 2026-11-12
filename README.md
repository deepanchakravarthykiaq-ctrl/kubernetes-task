# Kubernetes Nginx Practical Task

This beginner lab covers all 8 tasks in the assessment: Minikube/kubectl, Namespace,
Deployment, Service, ConfigMap, Secret, PV/PVC, scaling, rolling update, rollback,
verification, and GitHub submission.

## 1. Install tools (Ubuntu terminal)

Run these commands from **any directory**:

```bash
sudo apt-get update
sudo apt-get install -y curl wget apt-transport-https ca-certificates conntrack
```

Install kubectl:

```bash
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor --yes -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

Install Minikube:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64
```

Install Docker if it is not already installed:

```bash
sudo apt-get install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker "$USER"
newgrp docker
```

Start Minikube with Docker driver:

```bash
minikube start --driver=docker
kubectl config current-context
kubectl get nodes
kubectl version --client
```

If Docker permission is denied, close and reopen the terminal after adding your user to the docker group, then run `minikube start --driver=docker` again. Do not run `kubectl` with sudo.

## 2. Put the project in a working directory

Extract the ZIP, then run:

```bash
cd ~/kubernetes-nginx-task
pwd
ls -R
```

All commands below are run from this project directory unless noted.

## 3. Apply the YAML files

Apply in this order so dependencies exist before the Deployment:

```bash
kubectl apply -f manifests/01-namespace.yaml
kubectl apply -f manifests/02-configmap.yaml
kubectl apply -f manifests/03-secret.yaml
kubectl apply -f manifests/04-persistent-volume.yaml
kubectl apply -f manifests/05-persistent-volume-claim.yaml
kubectl apply -f manifests/06-deployment.yaml
kubectl apply -f manifests/07-service.yaml
```

Or apply the whole folder (Kubernetes accepts the namespace and namespaced resources in the same folder):

```bash
kubectl apply -f manifests/
```

## 4. Verify resources

```bash
kubectl get namespaces
kubectl get pv
kubectl get pvc -n devops-lab
kubectl get deployments -n devops-lab
kubectl get pods -n devops-lab -o wide
kubectl get services -n devops-lab
kubectl describe deployment nginx-deployment -n devops-lab
```

Wait for the Pods:

```bash
kubectl rollout status deployment/nginx-deployment -n devops-lab
```

Expected: 2/2 Pods ready, PVC Bound, and Service NodePort 30080.

## 5. Access Nginx

```bash
minikube service nginx-service -n devops-lab --url
```

Open the URL shown in your browser. If using a terminal-only environment, test it with:

```bash
curl "$(minikube service nginx-service -n devops-lab --url)"
```

If `minikube service --url` does not work in your environment, use:

```bash
minikube ip
curl http://$(minikube ip):30080
```

## 6. ConfigMap and Secret verification

```bash
kubectl describe configmap nginx-config -n devops-lab
kubectl describe secret nginx-secret -n devops-lab
kubectl exec deployment/nginx-deployment -n devops-lab -- printenv APP_ENV
kubectl exec deployment/nginx-deployment -n devops-lab -- printenv DB_USERNAME
```

Do not print or publish the Secret password in screenshots or GitHub. The sample password is for this local lab only; replace it for any real use. Kubernetes Secrets are base64-encoded by default, not automatically encrypted in every cluster.

## 7. Persistent storage verification

```bash
kubectl get pv
kubectl get pvc -n devops-lab
kubectl exec deployment/nginx-deployment -n devops-lab -- sh -c 'echo "Storage test" > /usr/share/nginx/html/storage/test.txt'
kubectl exec deployment/nginx-deployment -n devops-lab -- cat /usr/share/nginx/html/storage/test.txt
```

The PV uses Minikube node `hostPath` storage at `/mnt/data/nginx-pv`. This is suitable for this single-node training lab, not production multi-node storage.

## 8. Scale replicas (Task 6)

```bash
kubectl scale deployment nginx-deployment --replicas=3 -n devops-lab
kubectl get pods -n devops-lab
kubectl get deployment nginx-deployment -n devops-lab
```

## 9. Rolling update and rollback (Task 7)

Update the container image:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.28-alpine -n devops-lab
kubectl rollout status deployment/nginx-deployment -n devops-lab
kubectl rollout history deployment/nginx-deployment -n devops-lab
kubectl get pods -n devops-lab
```

Take a screenshot of the successful rollout. To demonstrate rollback:

```bash
kubectl rollout undo deployment/nginx-deployment -n devops-lab
kubectl rollout status deployment/nginx-deployment -n devops-lab
kubectl get deployment nginx-deployment -n devops-lab -o wide
```

The live Deployment image is changed by `kubectl set image`; the original YAML remains at `nginx:1.27-alpine`. If you re-apply the original Deployment YAML after the update, it will set the image back to the original version.

## 10. Final submission evidence (Task 8)

Run these and save screenshots or copy the output into your submission:

```bash
kubectl get pods -n devops-lab -o wide
kubectl get svc -n devops-lab
kubectl get deployments -n devops-lab
kubectl get pv
kubectl get pvc -n devops-lab
kubectl rollout history deployment/nginx-deployment -n devops-lab
```

## 11. Push to GitHub

Create an empty GitHub repository, for example `kubernetes-nginx-practical`. From the project directory:

```bash
git init
git add .
git commit -m "Complete Kubernetes Nginx practical tasks"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/kubernetes-nginx-practical.git
git push -u origin main
```

Replace `YOUR_USERNAME` with your GitHub username. If Git asks for a password over HTTPS, use a GitHub Personal Access Token (PAT), not your account password. Never commit a real token or production Secret.

## 12. Cleanup (only after screenshots/submission)

```bash
kubectl delete namespace devops-lab
kubectl delete pv nginx-pv
minikube stop
```

## Task-to-file mapping

| Assessment task | What to do / file |
|---|---|
| 1. Install Minikube and kubectl | Installation commands in this README |
| 2. Namespace and Nginx Deployment | `01-namespace.yaml`, `06-deployment.yaml` |
| 3. Expose Deployment with Service | `07-service.yaml` (NodePort) |
| 4. ConfigMap and Secret | `02-configmap.yaml`, `03-secret.yaml`, Deployment `envFrom` |
| 5. PV, PVC and mount | `04-persistent-volume.yaml`, `05-persistent-volume-claim.yaml`, Deployment volume mount |
| 6. Deployment YAML and scale | `06-deployment.yaml`, `kubectl scale` |
| 7. Rolling update and rollback | `kubectl set image`, `kubectl rollout undo` |
| 8. Verify and submit | Verification commands above; submit all files and screenshots |

## Important directory reminder

- **Outside project directory:** tool installation, `minikube start`, and cluster-wide checks can be run anywhere.
- **Inside `~/kubernetes-nginx-task`:** `kubectl apply -f manifests/`, Git commands, and commands using relative paths such as `manifests/06-deployment.yaml`.
- **Namespace:** use `-n devops-lab` for namespaced resources.
