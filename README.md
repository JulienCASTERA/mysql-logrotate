# Administration BDD - MYSQL Backups 

## TP1 

```
Créez une image Docker qui contient tous les outils nécessaires
pour mettre en place un système de backup automatique;
```

Pour ceci, on crée un fichier `Dockerfile` et un autre `docker-compose.yml`, le but est de partir du dockerfile de mysql de base et d'y installer cron.

Grâce au fichier docker-compose on va pouvoir exécuter notre build.

## TP2

```
Mettez en place une stratégie de backups grâce à cron qui génère un dump 
de la base de données tous les lundis à 17h et génère un fichier compressé 
en format gzip contenant la date de création.
```

Grâce au TP1 on va pouvoir rajouter une petite commande dans le dockerfile afin d'y insérer directement la crontab.

```bash
crontab -l | { cat; echo "0 17 * * 1 mysqldump -uroot -proot --all-databases | 
gzip -9 -c > /backups/all_databases_$(date +\%Y-\%m-\%d-\%H:\%M:\%S).sql.gz"; } | 
crontab -
```

Cette commande va récuperer (s'il y a déjà) les crontab existantes et rajouter celle-ci dans la liste.
Elle s'executera tous les lundis à `17H00` et fera un backup compressé.


## TP3

```
Mettez en place une stratégie de backups avec logrotate qui réalise un 
dump journalier compressé en format bz2 et qui garde les 5 derniers dumps.
```

Grâce à logrotate, on peut se passer de faire un mysqldump via la crontab pour mettre en place un système un peu plus sympathique.

Le but est de configurer logrotate afin qu'il fasse un dump de mysql journalier et faire une rotation sur 5 fichiers.

La configuration se fait grâce à ce fichier que l'on a copié dans notre container via le Dockerfile.

`COPY mysql /etc/logrotate.d/mysql`

```
/backups/all_databases {
  daily
  rotate 5
  compress
  compresscmd bzip2
  compressext .bz2
  create 640 mysql mysql
  postrotate
      mysqldump -uroot -proot --all-databases | gzip -9 -c > all_databases.sql.gz
  endscript
}
```

là ou `/backups/all_databases` est un fichier créé au préalable dans le dockerfile `touch /backups/all_databases`


A la fin, en ayant suivi les 3 TPs sur le même dockerfile, on a ET la crontab ET le logrotate, mais ceci est seulement à des fins de démonstration, en réalité il faudrait choisir soi l'un, soi l'autre, logrotate étant le plus attirant à mon sens.