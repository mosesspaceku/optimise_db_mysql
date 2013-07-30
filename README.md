#!/bin/bash
if [ "$1" == "" ]; then
        exit 1
        else
        DB_NAME=$1
fi

DB_USER="root"
DB_PASSWORD="password"
DB_HOST="10.1.49.200"
MYSQL="$(which mysql)"
MYSQL_LOGIN="${MYSQL} -h${DB_HOST} -u${DB_USER} -p${DB_PASSWORD} -Bse"
${MYSQL_LOGIN} "use ${DB_NAME}; show tables" > /tmp/${DB_NAME}.log
for table_name in `cat /tmp/${DB_NAME}.log`; do
ENGINE=`${MYSQL_LOGIN} "use ${DB_NAME}; show table status like '${table_name}'\G" | grep "Engine:" | awk '{print $2}'`
if [ ${ENGINE} == 'MyISAM' ];then
        for chk in --auto-repair --check --analyze --optimize; do
        mysqlcheck -h${DB_HOST} -u${DB_USER} -p${DB_PASSWORD} ${chk} ${DB_NAME} ${table_name}
        done
fi
done
