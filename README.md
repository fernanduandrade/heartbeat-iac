# Heartbeat IaC

Manifestos Kubernetes da `heartbeat-api` para sincronizacao via ArgoCD.

## Estrutura

```text
apps/heartbeat-api/                 # Manifests aplicados pelo ArgoCD
```

## Fluxo de deploy

1. Um push na `main` da `heartbeat-api` executa o workflow CI/CD.
2. O workflow gera e publica a imagem `DOCKERHUB_USERNAME/heartbeat-api:<short-sha>` no Docker Hub.
3. O workflow atualiza somente a imagem em `apps/heartbeat-api/kustomization.yaml`.
4. O ArgoCD detecta o commit neste repositorio e sincroniza o deployment.

Os demais manifestos continuam sendo a fonte da verdade neste repositorio. Alteracoes como replicas, probes, recursos, labels e service devem ser feitas aqui, sem serem sobrescritas pelo workflow da API.

Para cadastrar a aplicacao via CLI do ArgoCD:

```bash
argocd app create heartbeat-api \
  --repo https://github.com/fernanduandrade/heartbeat-iac.git \
  --revision main \
  --path apps/heartbeat-api \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace heartbeat \
  --sync-policy automated \
  --auto-prune \
  --self-heal
```

Depois de criada, o ArgoCD passa a observar `apps/heartbeat-api` e sincroniza quando houver alteracao na imagem ou nos manifestos.
