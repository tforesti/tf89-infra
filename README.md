# tf89-infra

Provisionne le VPS derrière `tf89.fr` et déploie les applications qui y tournent.

## Provisionner le VPS

```bash
cd ansible
ansible-playbook site.yml
```

Cibles partielles :

```bash
ansible-playbook site.yml --tags common    # Docker, UFW, dossiers /srv/*
ansible-playbook site.yml --tags nginx     # vhosts uniquement
```

## Déployer une app

Chaque application a un playbook dédié et ses variables dans `group_vars/` (fichiers `.example` à copier).

```bash
cd ansible
cp group_vars/<app>.yml.example group_vars/<app>.yml
cp group_vars/secrets.yml.example group_vars/secrets.yml
# éditer les variables

ansible-playbook deploy-<app>.yml
```

Relancer après un changement :

```bash
ansible-playbook deploy-<app>.yml
```

Premier déploiement :

1. DNS propagé vers le VPS
2. `certbot_email: ""` dans `group_vars/all.yml` → `ansible-playbook site.yml`
3. Déployer l'app
4. Renseigner `certbot_email`, relancer `ansible-playbook site.yml --tags nginx`

## Ajouter une app

Dans `group_vars/all.yml` :

```yaml
proxy_apps:
  - server_name: mon-app.tf89.fr
    upstream_port: 8080
  - server_name: autre-app.tf89.fr
    upstream_port: 8081
```

Puis `ansible-playbook site.yml --tags nginx`.
