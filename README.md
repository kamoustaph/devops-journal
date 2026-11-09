# Journal de bord DevOps

## Semaine 1 — Git et SSH

- `ssh-keygen -t ed25519` : crée une paire de clés SSH pour authentifier la machine auprès de GitHub.
- `cat ~/.ssh/id_ed25519.pub` : affiche la clé publique à ajouter au compte GitHub.
- `cd ~` : revient dans mon dossier personnel.
- `git clone git@github.com:kamoustaph/devops-journal.git` : copie le dépôt GitHub sur ma machine virtuelle.
- `cd ~/devops-journal` : ouvre le dossier du journal.
- `git add README.md` : prépare le journal pour le prochain commit.
- `git commit -m "Ajoute le journal de la semaine"` : enregistre une version du journal.
- `git push -u origin main` : envoie le commit sur GitHub.
- `git status` : vérifie l’état du dépôt.
