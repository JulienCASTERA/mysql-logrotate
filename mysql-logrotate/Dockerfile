FROM mysql:5.7.31

RUN apt-get update && \
    apt-get install logrotate -y && \
    apt-get install cron -y && \
    apt-get install vim -y && \
    crontab -l | { cat; echo "0 17 * * 1 mysqldump -uroot -proot --all-databases | gzip -9 -c > /backups/all_databases_$(date +\%Y-\%m-\%d-\%H:\%M:\%S).sql.gz"; } | crontab - && \
    touch /backups/all_databases

COPY mysql /etc/logrotate.d/mysql

EXPOSE 3306 33060