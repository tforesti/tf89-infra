# tf89-infra — guide agent

Provisionnement VPS OVH (`tf89.fr`) et déploiement des applications. **Repo séparé** des apps (ex. `copro-health-map`).

## Architecture VPS

```
Internet → Nginx (hôte, ports 80/443)
              ├── copro-health-map.tf89.fr → 127.0.0.1:8080 (docker compose)
              └── tf89.fr → /srv/sites/tf89.fr (statique)
```

Sur le VPS :

```
/srv/apps/<app>/     # clones Git des applications
/srv/sites/          # sites statiques
/srv/infra/          # clone optionnel de ce repo
```

## Playbooks

| Playbook | Rôle |
|----------|------|
| `ansible/site.yml` | Serveur : Docker, UFW, Nginx, Certbot |
| `ansible/deploy-copro.yml` | App copro-health-map : clone, `.env`, `docker compose up` |

## Fichiers clés

| Fichier | Rôle |
|---------|------|
| `ansible/inventory.yml` | IP VPS, user SSH (`debian`) |
| `ansible/group_vars/all.yml` | Domaines, ports proxy, `certbot_email` |
| `ansible/group_vars/copro_health_map.yml` | Repo Git, branche, URL app |
| `ansible/group_vars/secrets.yml` | Mots de passe (gitignored) |
| `ansible/roles/common/` | Docker (dépôt officiel), UFW, `/srv/*` |
| `ansible/roles/nginx/` | Vhosts + Certbot |
| `ansible/roles/copro_health_map/` | Clone, `.env`, compose, import RNC optionnel |

## Conventions

- Repo app en **HTTPS** pour les repos publics (pas de clé SSH GitHub sur le VPS)
- `POSTGRES_PASSWORD` ne peut pas être vide (`docker-compose.yml` impose une valeur)
- Import RNC : le CSV doit être **copié dans le conteneur** (`docker cp`) avant `import:rnc`
- Certbot : laisser `certbot_email` vide au premier run, le renseigner une fois l'app HTTP OK
- Convention ports locaux : 8080 = copro-health-map, 8081 = prochain projet

## Commandes

```bash
cd ansible
ansible-galaxy collection install -r requirements.yml
ansible-playbook site.yml --ask-become-pass
ansible-playbook deploy-copro.yml --ask-become-pass
ansible-playbook site.yml --tags nginx --ask-become-pass   # HTTPS
```

## Ne pas

- Mettre le code applicatif (Rails, React) dans ce repo
- Committer `inventory.yml`, `group_vars/all.yml`, `group_vars/secrets.yml` (gitignored)
- Utiliser `apt_repository` pour Docker — utiliser `deb822_repository` (Debian moderne)
