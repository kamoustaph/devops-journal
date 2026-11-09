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

## OPS-202 — Les outils de l’équipe

Objectif : installer htop pour observer les processus et ncdu pour analyser l’espace disque.

- `sudo apt update` : actualise la liste des paquets disponibles.
- `sudo apt install htop ncdu` : installe les deux outils d’observation.
- `dpkg -L ncdu` : affiche les fichiers installés par le paquet ncdu.
- `dpkg -L ncdu | grep '/bin/ncdu$'` : retrouve le binaire ncdu, installé dans /usr/bin/ncdu.
- `dpkg -s htop ncdu | grep -E '^(Package|Status):'` : vérifie l’état des deux paquets.

Résultat : htop et ncdu affichent tous les deux « install ok installed >>

## OPS-203 — L’API ne démarre plus

### Diagnostic et corrections

- `systemctl status mboa-api` : constate que le service est en échec avec le code 203/EXEC.
- `sudo journalctl -u mboa-api` : consulte les messages d’erreur du service.
- `sudo systemctl cat mboa-api` : affiche sa configuration.
- `ls -l /usr/bin/python /usr/bin/python3` : confirme que python est absent et que python3 existe.
- `sudo nano /etc/systemd/system/mboa-api.service` : corrige ExecStart en `/usr/bin/python3 api.py`.
- Après rechargement et relance, le journal montre l’erreur 200/CHDIR : le dossier de travail n’existe pas.
- Une seconde modification remplace `/srv/mboamarket/ap` par `/srv/mboamarket/app` dans WorkingDirectory.

### Relance et activation

Après chaque modification :
- `sudo systemctl daemon-reload` : recharge la configuration des services.
- `sudo systemctl reset-failed mboa-api` : remet à zéro le compteur des échecs.
- `sudo systemctl restart mboa-api` : relance l’API.

Puis :
- `sudo systemctl enable mboa-api` : active le service au démarrage de la machine.

### Vérifications

- `systemctl is-active mboa-api` : retourne active.
- `systemctl is-enabled mboa-api` : retourne enabled.
- `ps -p "$(systemctl show mboa-api -p MainPID --value)" -o pid,user,args` : confirme que Python tourne sous mboa.
- `curl http://127.0.0.1:8080/sante` : retourne OK Mboa Market API.

Bilan : les deux chemins incorrects ont été corrigés. L’API fonctionne sous mboa et son démarrage automatique est activé.
