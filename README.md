# 2048 Game on Kubernetes: Ingress-nginx to HAProxy Migration

This project demonstrates deploying the 2048 game on Kubernetes, exposing it via an Ingress resource using **ingress-nginx**, and then migrating to **ingress-haproxy**. It showcases how to switch Ingress controllers in a real cluster.

## Project Structure

- `2048GameInstall.yaml` — Namespace, Deployment, and Service for the 2048 game (namespace: `2048-namespace`, service port: 8084)
- `2048ingressnginx.yaml` — Ingress resource for ingress-nginx (`host: 2048-nginx.localtest.me`, service: svc-2048, port: 80)
- `2048ingresshaproxy.yaml` — Ingress resource for ingress-haproxy (`host: 2048-haproxy.localtest.me`, service: svc-2048, port: 8081)
- `ingress-nginx-controller-complete.yaml` — Full manifest for ingress-nginx controller
- `ingress-haproxy-controller-complete.yaml` — Full manifest for ingress-haproxy controller

## Prerequisites

- Kubernetes cluster (local with Docker-Desktop or cloud)
- `kubectl` installed and configured
- Cluster access to install Ingress controllers

## 1. Deploy the 2048 Game

Apply the combined manifest to create the namespace, deployment, and service:

```sh
kubectl apply -f 2048GameInstall.yaml
```

This creates:
- Namespace: `2048-namespace`
- Deployment: 2 replicas of the 2048 game (`public.ecr.aws/l6m2t8p7/docker-2048:latest`)
- ClusterIP Service: `2048-service` on port 8084

## 2. Expose with ingress-nginx

### Install ingress-nginx

You can use the provided manifest:

```sh
kubectl apply -f ingress-nginx-controller-complete.yaml
```

### Apply the Ingress resource

Edit `2048ingressnginx.yaml` if needed, then:

```sh
kubectl apply -f 2048ingressnginx.yaml
```

### Test access

- Add `2048-nginx.localtest.me` to your `/etc/hosts` pointing to your ingress controller’s IP (or use the Host header in curl).
- Example:
  ```sh
  curl -H "Host: 2048-nginx.localtest.me" http://<INGRESS_IP>:<PORT>
  ```

## 3. Migrate to ingress-haproxy

### Install ingress-haproxy

You can use the provided manifest:

```sh
kubectl apply -f ingress-haproxy-controller-complete.yaml
```

### Apply the HAProxy Ingress

Edit `2048ingresshaproxy.yaml` if needed, then:

```sh
kubectl apply -f 2048ingresshaproxy.yaml
```

### Test access

- Use the same host (`2048-haproxy.localtest.me`) and NodePort or LoadBalancer IP:
  ```sh
  curl -H "Host: 2048-haproxy.localtest.me" http://localhost:<haproxy-port>
  ```

## 4. Migration Notes

- Confirm that the application is running fine with the HAProxy controller, this should prove that the migration from ingress-NGINX controller to HAProxy controller was successful !!!. 
- Since, we only need one Ingress controller to manage a given Ingress resource (by `ingressClassName`).
- You can delete the nginx Ingress after migration:
  ```sh
  kubectl delete ingress ingress-2048
  kubectl delete -f ingress-nginx-controller-complete.yaml
  ```

## 5. Troubleshooting

- Check pod and service status:
  ```sh
  kubectl get pods -n 2048-namespace
  kubectl get svc -n 2048-namespace
  ```
- Check Ingress status:
  ```sh
  kubectl get ingress -n 2048-namespace
  kubectl describe ingress <name> -n 2048-namespace
  ```
- Check controller logs:
  ```sh
  kubectl -n ingress-nginx logs <nginx-pod>
  kubectl -n haproxy-controller logs <haproxy-pod>
  ```

## 6. Customization

- Change the host in the Ingress YAMLs to match your DNS setup.
- Add annotations for SSL, security headers, rate limiting, etc.

---
