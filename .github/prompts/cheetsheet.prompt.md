---
mode: agent
model: GPT-5 mini (copilot)
description: Cresates cheetsheet for the topics mentioned
---
# Cheatsheets

This file will hold compact, practical cheat sections for topics you revise. Each section should follow this pattern.

## Example: kubectl basics
Summary: kubectl is the Kubernetes CLI used to manage clusters.
Key commands:
- kubectl apply -f <file>
- kubectl get pods -o wide
- kubectl logs <pod>
Minimal example:
```bash
kubectl apply -f k8s/nginx-deploy.yaml
kubectl get pods
kubectl logs <pod-name>
```
Links:
- https://kubernetes.io/docs/reference/kubectl/overview/