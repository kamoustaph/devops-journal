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

## OPS-204 — Sauvegarde toutes les 5 minutes

### Objectif
Sauvegarder automatiquement le fichier /srv/mboamarket/data/commandes.csv toutes les 5 minutes et conserver une trace de chaque exécution.

### Script
- `sudo nano /usr/local/bin/mboa-sauvegarde` : crée le script de sauvegarde.
- Le script utilise `mkdir -p` pour créer /var/backups/mboamarket si nécessaire.
- `date +%Y%m%d-%H%M` : produit la date utilisée dans le nom commandes-AAAAMMJJ-HHMM.csv.
- `cp` : copie le fichier source vers la sauvegarde datée.
- `echo` : affiche un message de réussite, récupéré dans le journal systemd.
- `set -eu` : arrête le script en cas d’erreur ou de variable non définie.
- `umask 077` : réserve les nouveaux fichiers au propriétaire.
- `sudo chown root:root /usr/local/bin/mboa-sauvegarde` : attribue le script à root.
- `sudo chmod 750 /usr/local/bin/mboa-sauvegarde` : autorise son exécution par root et sa lecture/exécution par le groupe root.

### Service et timer
- `sudo nano /etc/systemd/system/mboa-sauvegarde.service` : crée le service Type=oneshot qui lance le script puis se termine.
- `systemd-analyze calendar '*-*-* *:00/5:00'` : vérifie le calendrier toutes les 5 minutes.
- `sudo nano /etc/systemd/system/mboa-sauvegarde.timer` : configure le déclenchement avec OnCalendar=*-*-* *:00/5:00 et AccuracySec=1s.
- `sudo systemctl daemon-reload` : recharge les fichiers de configuration systemd.
- `sudo systemctl enable --now mboa-sauvegarde.timer` : active le timer immédiatement et au démarrage.
- `sudo systemctl start mboa-sauvegarde.service` : teste une sauvegarde manuellement.

### Vérifications et résultats
- `systemctl is-active mboa-sauvegarde.timer` : retourne active.
- `systemctl is-enabled mboa-sauvegarde.timer` : retourne enabled.
- `systemctl list-timers --all mboa-sauvegarde.timer --no-pager` : confirme un passage à 17:20 puis le suivant à 17:25, le 9 novembre 2026.
- `sudo systemctl status mboa-sauvegarde.service --no-pager -l` : indique status=0/SUCCESS. L’état inactive (dead) est normal après la fin d’un service oneshot.
- `sudo journalctl -u mboa-sauvegarde.service -n 20 --no-pager` : montre les messages de sauvegarde de l’exécution manuelle à 17:18 et automatique à 17:20.
- `sudo ls -l /var/backups/mboamarket` : confirme la présence des deux copies datées, avec les permissions 600.
- `sudo cmp /srv/mboamarket/data/commandes.csv /var/backups/mboamarket/commandes-20261109-1720.csv` : confirme que la copie est identique


## OPS-205 — Transaction en échec

- `sudo journalctl -t mboa-paiement --no-pager -o short-iso` : consulte les messages du module de paiement.
- `sudo journalctl -t mboa-paiement -p err..err --no-pager -o short-iso` : sélectionne uniquement les messages de priorité erreur.
- Transaction identifiée : TX-95615.
- Date et heure : 7 novembre 2026 à 17:58:51 (+01:00).
- Cause signalée : absence de réponse du prestataire de paiement après 30 secondes. Le débit du client reste à confirmer.
- `printf '%s\n' 'TX-95615' > ~/incident-paiement.txt` : écrit uniquement le numéro demandé dans le fichier de preuve.
- `cat ~/incident-paiement.txt` : vérifie son contenu.

