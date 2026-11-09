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
## OPS-201 — Le serveur rame

### Diagnostic
- `top` et `ps aux --sort=-%cpu | head` ont identifié le programme `/usr/local/bin/mboa-miniatures`, qui consommait environ 60 % du CPU.
- PID : 2214.
- Utilisateur : mboa.
- `ps -p 2214 -o pid,user,lstart,etime,%cpu,args` a montré une durée d’exécution de 1 jour, 22 heures, 8 minutes et 47 secondes.

### Correction
- `sudo kill -TERM 2214` : demande l’arrêt propre du programme.
- `ps -p 2214 -o pid,user,stat,args` : ne montre plus le processus.
- `pgrep -af '[m]boa-miniatures'` : ne retourne aucun résultat.
- SIGKILL n’a pas été nécessaire.

### Vérification
- Avant l’arrêt, les charges moyennes sur 1 et 5 minutes étaient de 0,99 et 0,77.
- Après l’arrêt, elles sont descendues à 0,02 et 0,35.
