# Deploy automatico VPS (GitHub Actions)

El workflow ya esta en:

- `.github/workflows/deploy.yml`

## 1) Crear clave SSH para GitHub Actions (en tu Mac)

```bash
ssh-keygen -t ed25519 -C "gha-deploy-mbj" -f ~/.ssh/gha_deploy_mbj
```

## 2) Autorizar esa clave en el VPS (root)

```bash
ssh-copy-id -i ~/.ssh/gha_deploy_mbj.pub root@TU_VPS_IP
```

Si no tienes `ssh-copy-id`, pega manualmente el contenido de `~/.ssh/gha_deploy_mbj.pub` en:

- `/root/.ssh/authorized_keys`

## 3) Crear Secrets en GitHub

En tu repo: `Settings > Secrets and variables > Actions > New repository secret`

- `VPS_HOST` = IP o dominio del VPS
- `VPS_USER` = `root`
- `VPS_SSH_KEY` = contenido completo de `~/.ssh/gha_deploy_mbj` (clave privada)

## 4) Subir el workflow

```bash
git add .github/workflows/deploy.yml DEPLOY_AUTOMATICO.md
git commit -m "add automatic VPS deploy workflow"
git push origin main
```

## 5) Verificar

En GitHub `Actions`, ejecuta/espera `Deploy VPS`.

En VPS:

```bash
cd /home/webs/web/makebyjordan.cloud/public_html
git log -1 --oneline
```
