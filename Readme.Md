# Application Repository README (`Applications-CI-CD`)

This repository documents **frontend and backend application deployment** of the Expense Application using **Helm**, **Kubernetes**, **Docker/ECR**, and **Jenkins CI/CD**.

**[Application Repository](https://github.com/K-Basavaraj/Applications-CI-CD)**  
- Contains **Frontend** and **Backend** application code.  
- Includes Helm charts and Kubernetes manifests for deployment.  
- Has Jenkinsfile for building Docker images, pushing to **ECR**, and deploying to **EKS**.  

---

## Target Group Binding
- If the frontend container runs as a **non-root user**, it **cannot bind to port 80**, because system ports below 1024 require elevated privileges.  
- Therefore, we use **port 8080** for the frontend application.

### Ensure the following:
- `nginx.conf` listens on **port 8080** instead of 80.  
- The ALB target group **health check port** is set to **8080** in the **[Infra-CI/CD Repository](https://github.com/K-Basavaraj/Dev-CI-CD-Project)**.  
- The frontend **service target port** is configured as **8080** instead of 80.  

---

## Frontend Pod Auto-Recreation & Target Group Behavior

If a frontend pod is deleted, Kubernetes automatically recreates a new one to maintain the desired replica count.  
The new pod is automatically registered with the **ALB Target Group**, ensuring uninterrupted traffic flow and availability.

We use the **Target Group only for the frontend**, not for the backend, because user-facing requests first hit the frontend service.  
The backend is accessed internally by the frontend and does not require direct exposure to the ALB.

Although the `frontend-service.yaml` contains ingress-related details, the **actual Target Group Binding and listener configuration** are managed through **Terraform** under the **`60-ALB` module**.  
This ensures that all networking components — **ALB**, **listeners**, and **Target Groups** — are **centrally managed** for consistency, control, and security.

In the frontend Helm chart, we use a **TargetGroupBinding** (`apiVersion: elbv2.k8s.aws/v1beta1`) to connect the frontend service with the **Target Group** created by Terraform.

---

### Request Flow

**CDN → ALB → Ingress (Listeners & Target Group) → Kubernetes (Frontend & Backend Pods) → RDS**

```
### Verify Pod Status

```
kubectl config set-context --current --namespace=expense
kubectl get pods -o wide
NAME                        READY   STATUS    RESTARTS   AGE    IP           NODE
backend-76dfd857db-6znxq    1/1     Running   0          23m    10.0.11.82   ip-10-0-11-18.ec2.internal
backend-76dfd857db-9wgv6    1/1     Running   0          23m    10.0.12.71   ip-10-0-12-163.ec2.internal
frontend-846f76d848-bx2m4   1/1     Running   0          7m3s   10.0.11.99   ip-10-0-11-18.ec2.internal
frontend-846f76d848-v9nmt   1/1     Running   0          7m3s   10.0.12.36   ip-10-0-12-163.ec2.internal
```