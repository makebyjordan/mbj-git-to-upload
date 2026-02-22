# Guia completa: Pages CMS + GitHub + Deploy automatico a VPS

Esta guia deja tu web estatica (HTML/CSS/JS + JSON) conectada para:

- Editar contenido visualmente desde Pages CMS.
- Guardar cambios en GitHub (commit automatico).
- Desplegar automaticamente al VPS en cada push a `main`.

---

## 1) Estructura recomendada del proyecto

Proyecto estatico con archivos tipo:

- `index.html`, `blog.html`, `post.html`, etc.
- `blog.js`, `styles.css`, etc.
- `posts.json`, `projects.json`, `tech.json`, `navbar-links.json`

Sin backend obligatorio.

---

## 2) Configurar Pages CMS en el repo

Crear archivo `.pages.yml` en la raiz:

```yml
media:
  input: media
  output: /media

content:
  - name: posts
    label: Posts
    type: file
    path: posts.json
    format: json

  - name: projects
    label: Projects
    type: file
    path: projects.json
    format: json

  - name: tech
    label: Tech
    type: file
    path: tech.json
    format: json

  - name: navbar
    label: Navbar Links
    type: file
    path: navbar-links.json
    format: json
```

Subir a GitHub:

```bash
git add .pages.yml
git commit -m "add Pages CMS config"
git push origin main
```

Luego entrar en [Pages CMS](https://pagescms.org), conectar GitHub y seleccionar el repo.

---

## 3) Preparar script de deploy en VPS

En el VPS (como `root`):

```bash
cat > /root/deploy-mbj.sh << 'EOF'
#!/usr/bin/env bash
set -e
cd /home/webs/web/makebyjordan.cloud/public_html
git fetch origin main
git reset --hard origin/main
EOF

chmod +x /root/deploy-mbj.sh
```

Probar manualmente:

```bash
/root/deploy-mbj.sh
```

---

## 4) Crear workflow de GitHub Actions

Crear archivo `.github/workflows/deploy.yml`:

```yml
name: Deploy VPS

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Prepare SSH
        run: |
          mkdir -p ~/.ssh
          printf '%s\n' "${{ secrets.VPS_SSH_KEY }}" | tr -d '\r' > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519

      - name: Deploy on VPS
        run: |
          ssh -o StrictHostKeyChecking=accept-new "${{ secrets.VPS_USER }}@${{ secrets.VPS_HOST }}" "/root/deploy-mbj.sh"
```

Subir a GitHub:

```bash
git add .github/workflows/deploy.yml
git commit -m "add automatic VPS deploy workflow"
git push origin main
```

---

## 5) Crear clave SSH para GitHub Actions

En tu Mac:

```bash
ssh-keygen -t ed25519 -N "" -C "gha-deploy-mbj" -f ~/.ssh/gha_deploy_mbj
```

Instalar publica en VPS:

```bash
ssh-copy-id -i ~/.ssh/gha_deploy_mbj.pub root@TU_VPS_IP
```

Prueba directa:

```bash
ssh -i ~/.ssh/gha_deploy_mbj root@TU_VPS_IP "/root/deploy-mbj.sh"
```

Si este comando funciona, GitHub Actions tambien funcionara.

---

## 6) Configurar Secrets en GitHub (repo)

Ruta correcta:

- `Repo -> Settings -> Secrets and variables -> Actions`

Crear `Repository secrets`:

1. `VPS_HOST` = IP o dominio del VPS
2. `VPS_USER` = `root`
3. `VPS_SSH_KEY` = contenido completo de:

```bash
cat ~/.ssh/gha_deploy_mbj
```

Debe incluir:

- `-----BEGIN OPENSSH PRIVATE KEY-----`
- `-----END OPENSSH PRIVATE KEY-----`

---

## 7) Primera prueba del deploy

En GitHub:

- Ir a `Actions`
- Entrar en `Deploy VPS`
- `Run workflow` (branch `main`)

Debe salir en verde (`Success`).

---

## 8) Flujo diario de trabajo

1. Editas contenido en Pages CMS.
2. Pages CMS hace commit en `main`.
3. GitHub Actions se dispara.
4. El VPS hace `git fetch` + `git reset --hard origin/main`.
5. Web actualizada automaticamente.

---

## 9) Comandos utiles de verificacion

En VPS:

```bash
cd /home/webs/web/makebyjordan.cloud/public_html
git log -1 --oneline
git status
```

En GitHub:

- `Actions -> Deploy VPS` para ver historial y errores.

---

## 10) Errores comunes y solucion

### Error: `Permission denied (publickey,password)`

Causa: clave SSH incorrecta/no instalada para root.

Solucion:

1. Re-crear clave sin passphrase.
2. `ssh-copy-id` al VPS.
3. Probar `ssh -i ~/.ssh/gha_deploy_mbj root@IP "/root/deploy-mbj.sh"`.
4. Reemplazar `VPS_SSH_KEY` en GitHub.

### Error en paso `Prepare SSH`

Causa habitual: formato de clave mal pegado.

Solucion:

- Volver a pegar `VPS_SSH_KEY` completa.
- Asegurar que incluye BEGIN/END.

### Workflow no aparece

Comprobar que existe en repo:

- `.github/workflows/deploy.yml`

y que fue subido a `main`.

---

## 11) Seguridad recomendada (minimo)

- Usar usuario deploy en lugar de `root` (opcional, recomendado).
- Limitar acceso SSH por IP si tu VPS lo permite.
- Rotar claves SSH periodicamente.
- Revisar logs de Actions de vez en cuando.

---

## 12) Checklist final

- [ ] `.pages.yml` configurado.
- [ ] Pages CMS conectado al repo.
- [ ] `/root/deploy-mbj.sh` creado y ejecutable.
- [ ] `.github/workflows/deploy.yml` en `main`.
- [ ] Secrets `VPS_HOST`, `VPS_USER`, `VPS_SSH_KEY` creados.
- [ ] Run manual en Actions en verde.
- [ ] Deploy automatico funcionando tras editar en Pages CMS.

