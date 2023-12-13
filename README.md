# docker-postgres-upgrade 10 to 16

https://github.com/tianon/docker-postgres-upgrade

# Run

ls -la /data/postgres<BR>
 drwxr-xr-x 4  999 docker 4096 Jan 18 18:32 10<BR>
 drwxr-xr-x 3  999 docker 4096 Jan 18 18:22 16<BR>

cd /data/postgres/<BR>
docker run --rm  -v /data/postgres:/var/lib/postgresql sqldbapg/pg-upgrade:10-to-16 --link

Your installation contains extensions that should be updated<BR>
with the ALTER EXTENSION command.  The file <B>update_extensions.sql</B><BR>
when executed by psql by the database superuser will update these extensions.<BR>


Upgrade Complete<BR>
----------------<BR>
Optimizer statistics are not transferred by pg_upgrade.<BR>
Once you start the new server, consider running:<BR>
    /usr/lib/postgresql/16/bin/vacuumdb --all --analyze-in-stages<BR>

Running this script will delete the old cluster's data files:<BR>
    ./delete_old_cluster.sh

# Изменения
https://postgrespro.ru/docs/postgresql/11/release-11.html#id-1.11.6.19.4<BR>
pg_dump теперь выгружает и свойства базы данных, а не только её содержимое

https://postgrespro.ru/docs/postgresql/12/release-12.html#id-1.11.6.14.4<BR>
Ликвидация специальных столбцов oid<BR>
Удаление типов данных abstime, reltime и tinterval<BR>
Перенос параметров recovery.conf в postgresql.conf<BR>
В новых индексах btree максимальный размер записи индекса сокращён на 8 байт

https://postgrespro.ru/docs/postgresql/13/release-13.html#id-1.11.6.10.4<BR>
Переименование параметра конфигурации wal_keep_segments в wal_keep_size<BR>
Прекращение поддержки обновления неупакованных (созданных до версии 9.1) расширений

https://postgrespro.ru/docs/postgresql/14/release-14.html#id-1.11.6.6.4<BR>
Необходимо пересоздать пользовательские объекты, использующие некоторые встроенные функции для работы с массивами, в связи с изменением типов аргументов (Том Лейн)<BR>
Ликвидированы устаревшие операторы проверки включения @ и ~ для встроенных геометрических типов данных<BR>
Ликвидация операторов вычисления факториала ! и !!, а также функции numeric_fac()<BR>
Недопущение использования индексов GiST операторами включения (<@ и @>) из модуля intarray<BR>
Предоставлены предопределённые роли pg_read_all_data и pg_write_all_data позволяющие выдавать доступ сразу ко всем объектам базы (таблицам, представлениям и схемам) в режимах только для записи и только для чтения.

P.S.<BR>
<pre><code>
docker run --rm  -v /data/postgres:/var/lib/postgresql sqldbapg/pg-upgrade:10-to-16 --link
...
Performing Consistency Checks
-----------------------------
Checking cluster versions                                     ok
Checking database user is the install user                    ok
Checking database connection settings                         ok
Checking for prepared transactions                            ok
Checking for system-defined composite types in user tables    ok
Checking for reg* data types in user tables                   ok
Checking for contrib/isn with bigint-passing mismatch         ok
Checking for incompatible "aclitem" data type in user tables  ok
Checking for user-defined encoding conversions                ok
Checking for user-defined postfix operators                   ok
Checking for incompatible polymorphic functions               ok
Checking for tables WITH OIDS                                 ok
Checking for invalid "sql_identifier" user columns            ok
Creating dump of global objects                               ok
Creating dump of database schemas                             ok
Checking for presence of required libraries                   fatal
...

cat /data/postgres/16/data/pg_upgrade_output.d/20231109T083947.578/loadable_libraries.txt 
could not load library "$libdir/plperl": ERROR:  could not access file "$libdir/plperl": No such file or directory
In database: pglogger

\dL+"

  Name   |  Owner   | Trusted | Internal language |      Call handler      |       Validator        |          Inline handler          | Access privileges |         Description    
      
---------+----------+---------+-------------------+------------------------+------------------------+----------------------------------+-------------------+------------------------
------
 plperl  | postgres | t       | f                 | plperl_call_handler()  | plperl_validator(oid)  | plperl_inline_handler(internal)  |                   | PL/Perl procedural lang
uage
 plperlu | postgres | f       | f                 | plperlu_call_handler() | plperlu_validator(oid) | plperlu_inline_handler(internal) |                   | 
 plpgsql | postgres | t       | f                 | plpgsql_call_handler() | plpgsql_validator(oid) | plpgsql_inline_handler(internal) |                   | PL/pgSQL procedural lan
guage

DROP LANGUAGE plperlu;
DROP EXTENSION plperl;

SELECT oid::regprocedure, * FROM pg_catalog.pg_proc WHERE probin = '$libdir/plperl';
               oid                
----------------------------------
 plperlu_call_handler()
 plperlu_inline_handler(internal)
 plperlu_validator(oid)
</code></pre>

<pre><code>
SELECT oid::regprocedure, * FROM pg_catalog.pg_proc WHERE probin = '$libdir/plperl';
drop function pg_catalog.plperlu_call_handler();
drop function pg_catalog.plperlu_inline_handler(internal);
drop function pg_catalog.plperlu_validator(oid);
</code></pre>
