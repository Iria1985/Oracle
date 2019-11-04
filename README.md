# Oracle
## COMO MONTAR UNA BASE ORACLE CON LA TBO DESDE EL PRINCIPIO Y NO MORIR EN EL INTENTO
<pre>
su - usuarioaplicativo
op opuid usuarioaplicativo
</pre>
Nos ponemos en modo ksh : es importante saber que la tbo NO FUNCIONA en modo bash


Cargamos el start aplicativo: 
<pre>
. /users/mjs00/exploit/script/start
</pre>
Cargamos variables:
<pre>
export TMOUT=0
export TBEEXSCRIPT=/users/tbe00/exploit/script
export TBO_TBO_FILE_USER=/users/mjs00/exploit/data/tbo_repository.cfg
export ORACLE_HOME=/soft/ora1220/db
export ORACLE_SID=MJS
export PATH=$ORACLE_HOME/bin:$PATH
export UNXSAVE=/users99/mjs00/save
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
</pre>
Cargamos el start TBO:
<pre>
. /users/tbo06/exploit/script/start
</pre>

Creamos nuestro Archivo de configuración de la TBO:
<pre>
touch /users/msj00/exploit/data/tbo_repository.cfg
</pre>
Le damos los permisos correctos:
<pre>
chmod 600 /users/mjs00/exploit/data/tbo_repository.cfg
</pre>
Creamos la configuración para la base de datos:
<pre>
tbo new db MJS
</pre>
Esto nos rellena el archivo de configuración de la tbo /users/msj00/exploit/data/tbo_repository.cfg

Tenemos que editar este archivo. Para ello podemos hacerlo de 2 maneras diferentes:

* Vi /users/msj00/exploit/data/tbo_repository.cfg
* tbo edit

En este archivo se habrá la entrada DB diferentes:
* <DB:MJS>

En el cambiaremos las siguientes variables:
* BO_DB_ORACLE_HOME=/soft/ora1220  Pondremos la versión de Oracle que usemos
* TBO_DB_DIR_REP=/users99/mjs00/save   El path donde guardaremos los backups

Generamos  las etiquetas de CREBASE:
<pre>
tbo new crebase MJS MJS
</pre>
Tenemos que volver a editar el archivo tbo_repository.cfg. Para ello podemos hacerlo de 2 maneras diferentes:
* Vi /users/msj00/exploit/data/tbo_repository.cfg
* tbo edit

Modificamos las variables siguientes en la entrada CREBASE:
* <CREBASE:MJS>
```
<CREBASE:MJS>
#Modele crebase 12

__OPTIONS__
__/OPTIONS__

__PARAMETER__
DB_ALIAS=MJS
DB_PRD=mjs
DB_PRDOC=00
DB_AUTO_PASSWD=auto
DB_CHARACTERSET=AL32UTF8
DB_NCHARACTERSET=
DB_MAXDATAFILES=1024
DB_NB_REDOLOGS=3
DB_TIME_ZONE=
DB_AUDIT_ACTIVE=no
DB_AWR_SETTINGS=60480/60/30
DB_GATHER_PREF=
__/PARAMETER__

__INIT__
SGA_TARGET=512M
PGA_AGGREGATE_TARGET=110M
PROCESSES=200
DB_BLOCK_SIZE=8192
DB_FILES=1024
RECYCLEBIN=OFF
NLS_LANGUAGE=american
NLS_TERRITORY=america
NLS_LENGTH_SEMANTICS='CHAR'
SHARED_POOL_SIZE=100M
FAST_START_MTTR_TARGET=600
LOG_CHECKPOINTS_TO_ALERT=TRUE
ARCHIVE_LAG_TARGET=3600
__/INIT__
__SYSTEM__
#FS:TBS:file size:extend on:alloc.[AUTO|1M|blocksize]
/users2:SYSTEM:500M
/users2:SYSAUX:1000M
/users2:REDOLOG1:100M
/users3:REDOLOG2:100M
/users2:CTRL1
/users3:CTRL2
/users2:UNDO:1G
/users2:TEMP:1500M::64k
/users2:ARCHIVELOG
:EXPLOIT:10M:500M
__/SYSTEM__

__DATAPLACEMENT__
#FS:TBS:file size:extend on:alloc.[AUTO|1M|blocksize]
/users3:DA1:500M:1G
/users3:IX1:100M:200M
__/DATAPLACEMENT__

__TEMPPLACEMENT__
#FS:TBS:file size:extend on:alloc.[AUTO|1M|blocksize]
/users2:TM1:1500M
__/TEMPPLACEMENT__
__USERS__
#USER:MDP:TBS:TEMP:classe
MJS:qi4khals:DA1:TM1:INDUS
__/USERS__
</CREBASE:MJS>
```

Creamos el script crebase.sh de instalación de la base
<pre>
tbo crebase -k MJS
</pre>
Vamos al path correcto:
<pre>
cd /users/mjs00/exploit/data/crebase_MJS
</pre>
Ejecutamos el script con consola ksh:
<pre>
ksh crebase.sh
</pre>
Una vez terminada la creación de la base volvemos a modificar el archivo tbo_repository.cfg y añadimos las siguientes etiquetas:
```
<SCRIPT:MJS>
#alias:module:operation:TXTXXXX:args
DB_STATE:dbadm:state::-a open
DB_STOP:dbadm:stop::
DB_START:dbadm:start::
DB_STATS:dbadm:gatherjob::-m start
DB_SPACE:dbadm:spacetbs::-a "80 8000"
DB_BACKUP_INC0:rman:backupsa::-t  INC0+ARC
DB_BACKUP_ARC:rman:backupsa::-t  ARC
DB_REORG_IDX:dbadm:reorg_index::-m ONLINE
DB_EXPORT_FULL:dbpump:expdp::-P 4 -C -a "FULL=Y"
</SCRIPT:MJS>

<RMAN_BACKUP:MJS>
TBO_RMAN_BACKUP_TYPE=INC0+ARC
TBO_RMAN_BACKUP_COMPRESS=COMP
TBO_RMAN_BACKUP_DEGREE=1
TBO_RMAN_BACKUP_RETENTION=35
TBO_RMAN_BACKUP_REP_RACINE=/users99/mjs00/save/r/p
</RMAN_BACKUP:MJS>

<RMAN_RESTAURATION:MJS>
TBO_RMAN_RESTAURATION_REP_RACINE=$(tbo ostools -o get_rman_attr -t rep0 -a MJS00)
TBO_RMAN_RESTAURATION_DBPITR=$(tbo ostools -o get_rman_attr -t dbpitr -a MJS00)
TBO_RMAN_RESTAURATION_DEGREE=1
TBO_RMAN_RESTAURATION_PFILE=$(tbo ostools -o get_rman_attr -t init -a MJS00)
TBO_RMAN_RESTAURATION_CONTROLFILE=AUTO:$(tbo ostools -o get_rman_attr -t ctrl -a MJS00)
TBO_RMAN_RESTAURATION_DBID=$(tbo ostools -o get_rman_attr -t dbid -a MJS00)
</RMAN_RESTAURATION:MJS>

<AUTOCONNECT:MJS>
TBO_AUTOCONNECT_DB=MJS
TBO_AUTOCONNECT_SCRIPT=MJS
TBO_AUTOCONNECT_RMAN_BACKUP=MJS
TBO_AUTOCONNECT_RMAN_RESTAURATION=MJS
</AUTOCONNECT:MJS>
```
COMANDOS ORACLE

* Conectarse a la base: sqlplus usuario/pass
* Editar el archivo de configuración a través de la tbo: tbo edit
* Cargar las variables del start de la tbo:  . /users/tbo06/exploit/script/start
* Mostrar los comandos disponibles: tbo
* Buscar comando: tbo . Loquesea
* Muestra las etiquetas del archivo tbo_repository.cfg: tbo ls all
```
	AUTOCONNECT                   MJS
	CREBASE                       MJS
	DB                            MJS
	PWDTAB                        MJS
	RMAN_BACKUP                   MJS
	RMAN_RESTAURATION             MJS
	SCRIPT                        MJS
```	
* Conectar con la base con consola ksh: db connect MJS
	 db connect MJS
	(yval4em0) [MJS]>
* Listar base de datos: db ls tbs

db connect MJS
(yval4em0) [MJS]>db ls tbs

TBS   Ext.      Seg.    BlSz   Extent   Tbs Size       Free       Used Pct.        Max Pct.
St.  Tablespace Name          Type  Mgt.  Al. Mgt.     (k)      (k)    (in Mo)    (in Mo)    (in Mo) Used    (in Mo)  Max
---- ------------------------ ----- ----- --- ------ ----- -------- ---------- ---------- ---------- ---- ---------- ----
ON   EXPLOIT                  PERM  LOC.  A   AUTO         8       64              10            9             1        10           500      0
ON   MJSQDDA1            PERM  LOC.  A   AUTO          8       64            500          499          1         0          1,024     0
ON   MJSQDDF1            PERM  LOC.  U   AUTO           8   10,240         11           10            1         9           11         9
ON   MJSQDIX1              PERM  LOC.  A   AUTO           8       64           100           99          1          1            200       1
ON   MJSQDTM0           TEMP  LOC.  U   MANUAL     8       64          1,500      1,499        1          0         1,500     0
ON   MJSQDTM1           TEMP  LOC.  U   MANUAL     8      128         1,500      1,500        0          0         1,500     0
ON   MJSQDUD1            UNDO  LOC.  A   MANUAL     8       64         1,024        696        328       32      1,024    32
ON   SYSAUX                   PERM  LOC.  A   AUTO           8       64         1,000        756        244        24      1,000    24
ON   SYSTEM                   PERM  LOC.  A   MANUAL     8       64           500          121        379       76        500    76
                                                                    ---------- ---------- ----------      ----------
Tot:                                                                     6,145      5,189        956           7,259

	• Listar los archivos: db ls files

                                                                                           File Size           Next      Max
Tbs Name / File Class  Filename                                                TYPE   F_ID        Mo Auto        Mo       Mo CREATION_TIME
---------------------- ------------------------------------------------------- ----- ----- --------- ----- -------- -------- -------------------
EXPLOIT                /users2/mjs00/base/dbaqddax.dbf                         DATA      5        10 YES          0      500 25/04/2019 11:40:25
MJSQDDA1               /users3/mjs00/base/mjsqdda1_1.dbf                       DATA      6       500 YES          0    1,024 25/04/2019 11:44:19
MJSQDDF1               /users2/mjs00/base/mjsqddf1.dbf                         DATA      4        11 NO           0        0 25/04/2019 11:16:54
MJSQDIX1               /users3/mjs00/base/mjsqdix1_1.dbf                       DATA      7       100 YES          0      200 25/04/2019 11:44:20
MJSQDTM0               /users2/mjs00/base/mjsqdtm0_1.dbf                       TEMP      1     1,500 NO           0        0
MJSQDTM1               /users2/mjs00/base/mjsqdtm1_1.dbf                       TEMP      2     1,500 NO           0        0
MJSQDUD1               /users2/mjs00/base/mjsqdud_1.dbf                        DATA      3     1,024 NO           0        0 25/04/2019 11:16:53
SYSAUX                 /users2/mjs00/base/mjsqdsysaux_1.dbf                    DATA      2     1,000 NO           0        0 25/04/2019 11:16:51
SYSTEM                 /users2/mjs00/base/mjsqdsy_1.dbf                        DATA      1       500 NO           0        0 25/04/2019 11:16:48
[ CONTROL FILE    ]    /users2/mjs00/base/mjscf01.dbf                          CTRL
[ CONTROL FILE    ]    /users3/mjs00/base/mjscf02.dbf                          CTRL
[ ONLINE REDO LOG ]    /users2/mjs00/base/mjsrl11.dbf                          REDO              100
[ ONLINE REDO LOG ]    /users2/mjs00/base/mjsrl21.dbf                          REDO              100
[ ONLINE REDO LOG ]    /users2/mjs00/base/mjsrl31.dbf                          REDO              100
[ ONLINE REDO LOG ]    /users3/mjs00/base/mjsrl12.dbf                          REDO              100
[ ONLINE REDO LOG ]    /users3/mjs00/base/mjsrl22.dbf                          REDO              100
[ ONLINE REDO LOG ]    /users3/mjs00/base/mjsrl32.dbf                          REDO              100
                                                                                           ---------
Total                                                                                          6,745

	• Listar los usuarios: db ls users

USERNAME        PASSWORD         ACCOUNT_STATUS   PROFILE              DEF-TBS         TEMP-TBS        LOCK_DATE EXPIRY_DA PTIME
--------------- ---------------- ---------------- -------------------- --------------- --------------- --------- --------- ---------
CERBERE2008                      OPEN             PROFILE_PSA_LIGHT    MJSQDDF1        MJSQDTM0                            25-APR-19
MJS                              OPEN             PROFILE_PSA_LIGHT    MJSQDDA1        MJSQDTM1                            25-APR-19
OUTLN                            OPEN             PROFILE_PSA_LIGHT    SYSTEM          MJSQDTM0                            25-APR-19
SYS                              OPEN             PROFILE_PSA_LIGHT    SYSTEM          MJSQDTM0                            25-APR-19
SYSTEM                           OPEN             PROFILE_PSA_LIGHT    SYSTEM          MJSQDTM0                            25-APR-19
TBO                              OPEN             PROFILE_PSA_LIGHT    EXPLOIT         MJSQDTM0                            25-APR-19
ANONYMOUS                        LOCKED           PROFILE_PSA_LIGHT    SYSAUX          MJSQDTM0        25-APR-19           25-APR-19
APPQOSSYS                        LOCKED           PROFILE_PSA_LIGHT    SYSAUX          MJSQDTM0        25-APR-19           25-APR-19
AUDSYS                           LOCKED           PROFILE_PSA_LIGHT    MJSQDDF1        MJSQDTM0        25-APR-19           25-APR-19
DBSNMP                           LOCKED           PROFILE_PSA_LIGHT    SYSAUX          MJSQDTM0        25-APR-19           25-APR-19
DIP                              LOCKED           PROFILE_PSA_LIGHT    MJSQDDF1        MJSQDTM0        25-APR-19           25-APR-19
GSMADMIN_INTERN                  LOCKED           PROFILE_PSA_LIGHT    SYSAUX          MJSQDTM0        25-APR-19           25-APR-19
AL

GSMCATUSER                       LOCKED           PROFILE_PSA_LIGHT    MJSQDDF1        MJSQDTM0        25-APR-19           25-APR-19
GSMUSER                          LOCKED           PROFILE_PSA_LIGHT    MJSQDDF1        MJSQDTM0        25-APR-19           25-APR-19
ORACLE_OCM                       LOCKED           PROFILE_PSA_LIGHT    MJSQDDF1        MJSQDTM0        25-APR-19           25-APR-19
SYSBACKUP                        LOCKED           PROFILE_PSA_LIGHT    SYSTEM          MJSQDTM0        25-APR-19           25-APR-19
SYSDG                            LOCKED           PROFILE_PSA_LIGHT    SYSTEM          MJSQDTM0        25-APR-19           25-APR-19
SYSKM                            LOCKED           PROFILE_PSA_LIGHT    SYSTEM          MJSQDTM0        25-APR-19           25-APR-19
DBSFWUSER                        EXPIRED & LOCKED DEFAULT              SYSAUX          MJSQDTM0        25-APR-19 25-APR-19 25-APR-19
GGSYS                            EXPIRED & LOCKED DEFAULT              SYSAUX          MJSQDTM0        25-APR-19 25-APR-19 25-APR-19
REMOTE_SCHEDULE                  EXPIRED & LOCKED DEFAULT              MJSQDDF1        MJSQDTM0        25-APR-19 25-APR-19 25-APR-19
R_AGENT

SYS$UMF                          EXPIRED & LOCKED DEFAULT              SYSTEM          MJSQDTM0        25-APR-19 25-APR-19 25-APR-19
SYSRAC                           EXPIRED & LOCKED DEFAULT              SYSTEM          MJSQDTM0        25-APR-19 25-APR-19 25-APR-19
XDB                              EXPIRED & LOCKED DEFAULT              SYSAUX          MJSQDTM0        25-APR-19 25-APR-19 25-APR-19
XS$NULL                          EXPIRED & LOCKED DEFAULT              SYSTEM          MJSQDTM0        25-APR-19 25-APR-19 25-APR-19

	• Chequear el estado de la base: db check
Database State                                     open                      OK
Instance Open Mode                          allowed                  OK
Database LOGMODE                           ARCHIVELOG        OK
Nb Indexs UNUSABLE                           0                            OK
Scan last 24h in Alert.log                    No errors              OK
Listener listening on port                    1521                      OK


	• Mostrar aletas de la base: db alert
	• tbo sql


HACER UN BACKUP DE UNA BASE DE DATOS

