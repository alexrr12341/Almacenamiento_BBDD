## Práctica Cooperativa (Almacenamiento)




Parte grupal
------------



ORACLE:

1. Cread un índice para la tabla EMP de SCOTT que agilice las consultas por nombre de empleado en un tablespace creado específicamente para índices. ¿Dónde deberiáis ubicar el fichero de datos asociado? ¿Cómo se os ocurre que podriáis probar si el índice resulta de utilidad?


Creamos el tablespace que tendrá la ubicación por defecto de todos los tablespace
```sql
Create tablespace Tablespace_Index
datafile '/opt/oracle/oradata/orcl/index.dbf'
size 5M
autoextend on;
```
Ahora creamos un indice que apunte a los nombres de los empleados y al tablespace creado
```sql
Create index emp_ename on scott.emp(ename)
Tablespace Tablespace_Index;
```

El fichero de datos asociado se tendría que ubicar en el directorio /opt/oracle/oradata/orcl, ya que este es la ruta por defecto de los datafiles.

Para comprobar si el índice resulta de utilidad, podemos realizar una consulta de la columna ename de la tabla emp y mostrar el tiempo que tarda en resolver la consulta con el parámetro set timing on; luego realizar la misma consulta pero con el índice definido.

###### Antes de crear el índice
```sql
select ename from emp;

ENAME
----------
CLARK
KING
MILLER
SMITH
JONES
SCOTT
ADAMS
FORD
ALLEN
WARD
MARTIN
BLAKE
TURNER
JAMES

14 filas seleccionadas.

Transcurrido: 00:00:00.01
```

###### Después de crear el índice
```sql
select ename from emp;

ENAME
----------
CLARK
KING
MILLER
SMITH
JONES
SCOTT
ADAMS
FORD
ALLEN
WARD
MARTIN
BLAKE
TURNER
JAMES

14 filas seleccionadas.

Transcurrido: 00:00:00.00
```

Como podemos apreciar, el tiempo es menor en la segunda consulta pero no es una gran diferencia ya que los registros que tiene dicha tabla no son abundantes para que la consulta muestre un rendimiento considerable con el índice respecto a sín el.


       
2. Realizad una consulta al diccionario de datos que muestre qué  índices existen para objetos pertenecientes al esquema de SCOTT y sobre qué columnas están definidos. Averiguad en qué fichero o ficheros de datos se encuentran las extensiones de sus segmentos correspondientes.

```sql
SELECT a.TABLE_NAME, a.INDEX_NAME, a.COLUMN_NAME, c.FILE_NAME
FROM DBA_IND_COLUMNS a, DBA_EXTENTS b, DBA_DATA_FILES c
WHERE a.TABLE_NAME = b.SEGMENT_NAME
AND b.FILE_ID = c.FILE_ID
AND a.TABLE_OWNER='SCOTT';

TABLE_NAME     INDEX_NAME   COLUMN_NAM     FILE_NAME
------------ ------------ -------------  ----------------------------------------
DEPT         PK_DEPT      DEPTNO         /opt/oracle/oradata/orcl/users01.dbf
EMP             PK_EMP          EMPNO          /opt/oracle/oradata/orcl/users01.dbf
EMP             EMP_ENAME      ENAME          /opt/oracle/oradata/orcl/users01.dbf

```
      

3. Cread una secuencia para rellenar el campo deptno de la tabla dept de forma coherente con los datos ya existentes. Insertad al menos dos registros haciendo uso de la secuencia.

Creamos la secuencia, que empezará por 50, y incrementará su número 10 veces ya que es como funciona deptno.
```sql
create sequence deptnosec
  start with 50
  increment by 10
  maxvalue 100
  minvalue 10;
```

Los valores de la secuencia podrá ser currval y nextval, currval servirá para poder repetir el valor ya dado en la secuencia, y nextval será el siguiente en la lista.
```sql
insert into dept values
  (deptnosec.nextval,'DEVOPS','SEVILLA');

insert into dept values
  (deptnosec.nextval,'DEVELOPER','CADIZ');
```

```sql
SQL> select * from dept;

    DEPTNO DNAME      LOC
---------- -------------- -------------
    10 ACCOUNTING      NEW YORK
    20 RESEARCH      DALLAS
    30 SALES      CHICAGO
    40 OPERATIONS      BOSTON
    50 DEVOPS      SEVILLA
    60 DEVELOPER      CADIZ

6 filas seleccionadas.
```
  

4. Queremos limpiar nuestro fichero tnsnames.ora. Averiguad cuales de sus entradas se están usando en algún enlace de la base de datos.

El fichero tsnames son las conexiones a las que nuestro servidor de Oracle puede acceder, hemos observado que hay una conexión activa.

```sql
SQL> select * from v$dblink;

DB_LINK            OWNER_ID LOG HET PROTOC OPENCURSORS IN UPD COMMIT_POINT_STRENGTH     CON_ID
-------------------- ---------- --- --- ------ ------------ --- --- --------------------- ----------
GRUPAL                  0 YES NO    UNKN          0 YES NO            0       1
```


Hay creados todos estos db_links, pero solo está activo uno, por lo que los demás podriamos descartarlos
```sql
SQL> select * from dba_db_links;

OWNER             DB_LINK          USERNAME           HOST            CREATED
-------------------- -------------------- -------------------- -------------------- --------
SYS             FERNANDO_DELEGADO      LUIS               orcluis            28/01/20
SYS             FERNANDO_LINKLUIS3   LUIS               orcluis            13/01/20
SYS             FERNANDO_LINKLUIS5   LUIS               orcluis            13/12/19
SYS             FERNANDO_LINKLUIS2   LUIS               orcl2            13/12/19
SYS             FERNANDO_LINKLUIS      LUIS               orcl            13/12/19
SYS             FERNANDO_LINKORCL      PALOMA           orcl            12/12/19
SYS             FERNANDO_LINK      PALOMA           oraDB            12/12/19
PUBLIC             GRUPAL          delegadofer           PSQLU            28/01/20
PUBLIC             FERNANDODELEGADOLUIS clienteconexion      PSQLU            28/01/20
PUBLIC             FERNANDODELEGADO      clienteconexion      PSQLU            28/01/20
PUBLIC             FERNANDOPOSTGRESLUIS clienteconexion      PSQLU            13/12/19

11 filas seleccionadas.
```


5. Meted las tablas EMP y DEPT de SCOTT en un cluster.

Creamos el tablespace para usarlo en el cluster
```sql
Create tablespace Cluster_empdept
datafile '/opt/oracle/oradata/orcl/empdept.dbf'
size 5M
autoextend on;
```

Creamos el cluster emp_dept que contiene deptno(su columna común)
```sql
CREATE CLUSTER emp_dept
(deptno NUMBER(2))
Tablespace Cluster_empdept;
```

Creamos el indice que usará el cluster

```sql
CREATE INDEX index_empdept ON CLUSTER emp_dept;
```

Creamos un backup de las tablas para después poder crear las tablas sin problema de datos 
```sql
CREATE TABLE SCOTT.DEPT_BAK
(
 DEPTNO NUMBER(2),
 DNAME VARCHAR2(14),
 LOC VARCHAR2(13),
 CONSTRAINT PK_DEPT_BAK PRIMARY KEY (DEPTNO)
);

CREATE TABLE SCOTT.EMP_BAK
(
 EMPNO NUMBER(4),
 ENAME VARCHAR2(10),
 JOB VARCHAR2(9),
 MGR NUMBER(4),
 HIREDATE DATE,
 SAL NUMBER(7, 2),
 COMM NUMBER(7, 2),
 DEPTNO NUMBER(2),
 CONSTRAINT FK_DEPTNO_BAK FOREIGN KEY (DEPTNO) REFERENCES SCOTT.DEPT_BAK (DEPTNO),
 CONSTRAINT PK_EMP_BAK PRIMARY KEY (EMPNO)
);
```

Insertamos los datos en la tabla backup
```sql
insert into SCOTT.EMP_BAK
SELECT * FROM SCOTT.EMP;

insert into SCOTT.DEPT_BAK
Select * from SCOTT.DEPT;

drop table SCOTT.EMP;

drop table SCOTT.DEPT;
```

Y creamos las tablas con el cluster insertado
```sql
CREATE TABLE SCOTT.DEPT
(
 DEPTNO NUMBER(2),
 DNAME VARCHAR2(14),
 LOC VARCHAR2(13),
 CONSTRAINT PK_DEPT PRIMARY KEY (DEPTNO)
)
Cluster emp_dept (deptno);

CREATE TABLE SCOTT.EMP
(
 EMPNO NUMBER(4),
 ENAME VARCHAR2(10),
 JOB VARCHAR2(9),
 MGR NUMBER(4),
 HIREDATE DATE,
 SAL NUMBER(7, 2),
 COMM NUMBER(7, 2),
 DEPTNO NUMBER(2),
 CONSTRAINT FK_DEPTNO FOREIGN KEY (DEPTNO) REFERENCES SCOTT.DEPT (DEPTNO),
 CONSTRAINT PK_EMP PRIMARY KEY (EMPNO)
)
Cluster emp_dept (deptno);
```

Y volvemos a insertar los datos y borramos las tablas backups
```sql
insert into SCOTT.EMP
SELECT * from SCOTT.EMP_BAK;

insert into SCOTT.DEPT
SELECT * FROM SCOTT.DEPT_BAK;

DROP TABLE SCOTT.EMP_BAK;
DROP TABLE SCOTT.DEPT_BAK;
```

Miramos que el cluster está en el diccionario de datos.
```sql
SQL> select cluster_name from dba_clusters where owner = 'SCOTT';

CLUSTER_NAME
--------------------------------------------------------------------------------------------------------------------------------
EMP_DEPT
```

6. Realizad un procedimiento llamado BalanceoCargaTemp que balancee la carga de usuarios entre los tablespaces temporales existentes. Para ello averiguará cuántos existen y asignará los usuarios entre ellos de forma equilibrada. Si es necesario para comprobar su funcionamiento, crea tablespaces temporales nuevos.

Vamos a observar cuantos tablespaces temporales tenemos

```sql
SQL> SELECT tablespace_name FROM dba_tablespaces WHERE contents = 'TEMPORARY';

TABLESPACE_NAME
------------------------------
TEMP

```

Como solo hay uno vamos a crear varios tablespaces

```sql
Create temporary tablespace custom_temp
tempfile '/opt/oracle/oradata/orcl/customtemp.dbf'
size 5M
autoextend on;

Create temporary tablespace custom_temp2
tempfile '/opt/oracle/oradata/orcl/customtemp2.dbf'
size 3M
autoextend on;


Create temporary tablespace custom_temp4
tempfile '/opt/oracle/oradata/orcl/customtemp4.dbf'
size 3M
autoextend on;

Create temporary tablespace custom_temp5
tempfile '/opt/oracle/oradata/orcl/customtemp5.dbf'
size 3M
autoextend on;

Create temporary tablespace custom_temp6
tempfile '/opt/oracle/oradata/orcl/customtemp6.dbf'
size 3M
autoextend on;
```


```sql
create or replace procedure ConseguirMinTablespace(p_tablespace IN OUT VARCHAR2,
                   p_cuenta IN OUT NUMBER)
is
begin

    Select count(u.username) as cuenta, t.tablespace_name INTO p_cuenta, p_tablespace
    from dba_users u, dba_tablespaces t
    where u.temporary_tablespace(+) = t.tablespace_name
    and t.contents = 'TEMPORARY'
    group by t.tablespace_name
    order by cuenta asc
    fetch first row only;

end;
/

create or replace procedure ConseguirMaxTablespace(p_tablespace IN OUT VARCHAR2,
                   p_cuenta IN OUT NUMBER)
is
begin

    Select count(u.username) as cuenta, t.tablespace_name INTO p_cuenta, p_tablespace
    from dba_users u, dba_tablespaces t
    where u.temporary_tablespace(+) = t.tablespace_name
    and t.contents = 'TEMPORARY'
    group by t.tablespace_name
    order by cuenta desc
    fetch first row only;

end;
/


create or replace procedure BalanceoCargaTemp
is

    v_numusuarios NUMBER;
    v_menor NUMBER;
    v_mayor NUMBER;
    v_menorts VARCHAR2(20);
    v_mayorts VARCHAR2(20);

begin
    ConseguirMinTablespace(v_menorts,v_menor);
    ConseguirMaxTablespace(v_mayorts,v_mayor);
    while v_mayor != v_menor loop
        if v_menor < v_mayor then
            v_numusuarios:=v_mayor - v_menor;
            if v_numusuarios = 1 then
                exit;
                else
                RealizarBalance(v_mayorts,v_menorts);
                ConseguirMinTablespace(v_menorts,v_menor);
                ConseguirMaxTablespace(v_mayorts,v_mayor);
            end if;
        end if;
    end loop;

end;
/

create or replace procedure RealizarBalance(p_tablespacemayor VARCHAR2,
                        p_tablespacemenor VARCHAR2)
is

    v_user VARCHAR2(50);

begin

    select username into v_user
    from dba_users
    where temporary_tablespace = p_tablespacemayor
    and username != 'XS$NULL'
    fetch first row only;
    execute immediate 'alter user '||v_user||' temporary tablespace '||p_tablespacemenor;

end;
/
```


7. Realizad un pequeño artículo o una entrada para un blog técnico explicando las limitaciones que presentan MySQL y Postgres para gestionar el almacenamiento de los datos respecto a ORACLE, si es que presentan alguna.

# Almacenamiento en Postgresql, MySQL y ORACLE
En este post se van a desarrollar las principales diferencias que hay entre los tres gestores de bases de datos en cuanto a la gestión del almacenamiento.

### PostgreSQL

Principales definiciones:

- **Bloques**: unidad más pequeña del disco que forman páginas.
- **Páginas**: almacena los datos, que a su vez se guardan en diferentes posiciones del disco. La dispersión de estas páginas es un gran problema para las DBMS. El tamaño mínimo, y por defecto, de una página es de 8k y máximo 32k. No se permiten tuplas más grandes que las páginas. 
- **Storage-toast**: técnica que utiliza para superar la limitación de que las tuplas de gran tamaño. Consiste en gruardar estos datos comprimidos y divididos en filas en una tabla paralela sin que el usuario lo llegue a apreciar. 
- **Buffer Manager**: controlador de la memoria.
- **Storage Manager**: responsable de la administración general de almacenamiento de los datos. PostgreSQL solo posee uno para todo el servidor compuesto por varios módulos que administran las transacciones y acceso a los objetos de las bases de datos. Los módulos son: **Transaction System**, **Relational Storage**, **Time Management**, **Concurrency Control**, **Timestamp Management** y **Record Acces**.
- **Tablespaces**: permite definir ubicaciones en el sistema de archivos donde almacenar los archivos que representan los objetos de la base de datos. 

Principales incovenientes en almacenamiento con respecto a ORACLE:

- Antes de la versión 9.0 no soportaba tablespaces para difinir dónde almacenar la base de datos, el esquema, los índices, etc.
- No presenta clausulas de almacenamiento de datos, lo que impide asignar cuotas de almacenamiento.
> Para controlar el espacio utilizado en tabla se puede utilizar la función **pg_total_ralation_size**.
- Las transacciones se abortan completamente en caso de fallo durante su ejecución.
- Tamaño máximo de una tabla: 32 TB.
- Tamaño máximo de una tupla: 1,6 TB.
- Tamaño máximo de un campo: 1 GB.

### MySQL

Principales definiciones:
- **Motor de almacenamiento**: se encarga de almacenar, manajar y recuperar información de un tabla. Los más conocidos son **MyISAM** e **InnoDB**.
- **MyISAM**: motor de almacenamiento por defecto. Tiene mayor velocidad en general para recuperar datos que InnoDB.
- **InnoDB**: motor de almacenamiento transaccional con capacidades de commit, rollback, recuperación de fallos, bloqueo de registros y características ACID. Indicado para usos elevados de INSERT y UPDATE.
> Para crear una tabla InnoDB se debe especificar en su creación:
~~~
CREATE TABLE customers (a INT, b CHAR (20), INDEX (a)) ENGINE=InnoDB;
CREATE TABLE customers (a INT, b CHAR (20), INDEX (a)) TYPE=InnoDB;
~~~

Principales incovenientes en almacenamiento con respecto a ORACLE:
- Mayor número de índices por tabla: 64 - ilimitado en ORACLE
- Número máximo de columnas usadas en índices 16 columnas - 30 en ORACLE
- Tamaño máximo de índice: 1000 Bytes.
- Máximo de registros soportados: 50 millones de registros - 450 mil millones en ORACLE.

https://mundodelinux.wordpress.com/2019/01/22/limitaciones-de-mysql-y-postgresql-en-la-gestion-de-almacenamiento-respecto-a-oracle-database/

https://openbinary20.com/2017/01/31/limitaciones-de-postgresql-y-mysql-respecto-a-oracle-en-gestion-de-almacenamiento/

NOTAS: Hay que ver si en MySQL hay tablespaces. Acabar las definiciones y poner los incovenientes.


8. Explicad en qué consiste el sharding en MongoDB.

Sharding consiste en repartir los conjuntos de datos a los distintos clusters, que denominamos shars, de tal forma que exista una distribución de los datos equitativamente a lo largo de los distintos nodos. Por lo tanto el sharding nos permite mejorar el rendimiento de nuestro sistema ya que al distribuir la carga entre distintas máquinas no existe sobrecarga de un nodo.

Elementos de la arquitectura

  -  Mongos: Son las instancias que hacen de interfaz entre los shards donde se encuentran los datos.
    
  -  Shards: Son los clusters en donde se encuentra los documentos de nuestras colecciones. Cada cluster contiene un subconjunto del dataset. La información dentro de un cluster se agrupa en torno a agrupaciones lógicas denominados chunks.

  -  Config Server: Son instancias mongod que contienen información asociados al dataset. En concreto indican donde se encuentran los documentos en base a shards o chunks. Puede desplegarse en forma de réplica.

Para que la información se distribuya entre los clusters :

    Split Chunks: Es la subdivisión de chunks y se produce cuando se sobrepasa el tamaño, ya que hay un límite de documentos.
    Migration Chunks: La migración es el procedimiento de distribuir un chunk de un shard a otro. Este proceso lo realiza un proceso llamado balancer que se encuentra en el nodo primario del Config Server. Cuando dicho proceso compara entre el número de chunks del shard que mas tiene con el que tiene menos, se inicia la migración.

El sharding trabaja a nivel de colección, con esto podemos decidir que colecciones queremos distribuir. Las colecciones que no hayamos distribuido se encontrarán en el shard primario.

![](/fotos/sharding.jpg)

9. Resolved el siguiente caso práctico en ORACLE:

En nuestra empresa existen tres departamentos: Informática, Ventas y Producción. En Informática trabajan tres personas: Pepe, Juan y Clara. En Ventas trabajan Ana y Eva y en Producción Jaime y Lidia.

a)  Pepe es el administrador de la base de datos.
Juan y Clara son los programadores de la base de datos, que trabajan tanto en la aplicación que usa el departamento de Ventas como en la usada por el departamento de Producción.
Ana y Eva tienen permisos para insertar, modificar y borrar registros en las tablas de la aplicación de Ventas que tienes que crear, y se llaman Productos y Ventas, siendo propiedad de Ana.
Jaime y Lidia pueden leer la información de esas tablas pero no pueden modificar la información.
Crea los usuarios y dale los roles y permisos que creas conveniente.
Creamos al usuario Pepe, que va a ser el Adminsitrador, por eso le asignamos el rol DBA. El rol DBA tiene el privilegio de ver y manejar todos los datos de la BD.
```sql
create user Pepe identified by admin;
grant DBA to Pepe;
```

Vamos a crear al usuario Juan y Clara, que van a ser los programadores.
```sql
create user Juan identified by juan;
create user Clara identified by clara;
```

Ahora vamos a crear un rol para los programadores, ya que van a desarrolar aplicaciones en la base de datos, para que puedan desarrolar aplicaciones deben de poder crear, modificar y elimnar los objetos de ellas. Los permisos que vamos a asignar a los programadores son:

|Privilegio         |Descripción
|:------------------|:---------------------
|CREATE TABLE       |
|CREATE VIEW        |
|CREATE PROCEDURE   |
|CREATE TRIGGER     |
|CREATE SEQUENCE    |
|CREATE SYNONYM     |

```sql
create role Programadores;
grant create any table, create any index, create view, create procedure, create trigger, create sequence, create synonym to Programadores;
grant connect to Programadores;
grant Programadores to Juan;
grant Programadores to Clara;
```

```sql
create user Ana identified by ana;
create user Eva identified by eva;
```

Para que Ana tenga los permisos, debemos crear las tablas como propiedad de Ana.

```sql
create table ANA.PRODUCTOS
(
Codigo VARCHAR2(10),
Nombre VARCHAR2(20),
Descripcion VARCHAR2(50),
Precio NUMBER,
constraint pk_codigop PRIMARY KEY(Codigo)
);

create table ANA.VENTAS
(
Codigo VARCHAR2(10),
CodigoProducto VARCHAR2(10),
CantidadVendida NUMBER,
Fecha DATE,
Lugar VARCHAR2(20),
constraint pk_codigov PRIMARY KEY(Codigo),
constraint fk_codigoprod FOREIGN KEY(CodigoProducto) REFERENCES ANA.PRODUCTOS
);
```

Creamos el rol ventas para darle los privilegios a los usuarios Ana y Eva.

```sql
CREATE ROLE VENTAS;
grant insert, delete, update on ANA.productos to Ventas;
grant insert, delete, update on ANA.ventas to Ventas;
grant Ventas to Ana;
grant Ventas to Eva;

```

```sql
insert into ANA.Productos
values('123123123L','Pollo','Pollo envasado','5');
insert into ANA.Productos
values('124124124M','Jamon','Jamon al vacio','10');
insert into ANA.Productos
values('125125125N','Chorizo','Chorizo iberico','8');

insert into ANA.Ventas
values('9768758901','123123123L','250','23-09-2019','Sevilla');
insert into ANA.Ventas
values('5836249231','124124124M','300','14-11-2019','Cordoba');
insert into ANA.Ventas
values('9843626321','125125125N','150','20-12-2019','Huelva');
```

También crearemos los usuarios Jaime y Lidia.

```sql
create user Jaime identified by jaime;
create user Lidia identified by lidia;
```


Utilizamos `GRANT READ` Bloquea los registros que devuelve el **SELECT** para que no puedan ser modificados, los registros que se le aplique el `for update`, por otras secciones.
Para que este bloqueo no se pueda realizar, tenemos utilizar el privilegio de `GRANT READ`, en vez de `GRANT SELECT`.
```sql
create role Produccion;
grant connect to Produccion;
grant read on ANA.PRODUCTOS to Produccion;
grant read on ANA.VENTAS to Produccion;
grant Produccion to Jaime;
grant Produccion to Lidia;
```

    b) Los espacios de tablas son System, Producción (ficheros prod1.dbf y prod2.dbf) y Ventas (fichero vent.dbf).
Los programadores del departamento de Informática pueden crear objetos en cualquier tablespace de la base de datos, excepto en System.
Los demás usuarios solo podrán crear objetos en su tablespace correspondiente teniendo un límite de espacio de 30 M los del departamento de Ventas y 100K los del de Producción.
Pepe tiene cuota ilimitada en todos los espacios, aunque el suyo por defecto es System.

##### TABLESPACE SYSTEM ya existe.

##### Si un tablespace está formado por 2 datafiles y tenemos una tabla en ese tablespace, a medida que vamos insertando filas éstas se almacenarán en cualquiera de los dos datafiles indistintamente, es decir, unas pueden estar en un datafile y otras en otro.


Crearemos ambos tablespaces para poder poner las cuotas a ambos usuarios
```sql
create tablespace Produccion
datafile '/opt/oracle/oradata/orcl/prod1.dbf' size 50M,
'/opt/oracle/oradata/orcl/prod2.dbf' size 50M
autoextend off;

create tablespace Ventas
datafile '/opt/oracle/oradata/orcl/vent.dbf' size 100M
autoextend off;
```
Añadiremos las cuotas a los usuarios, en este caso Ana y Eva tendran el límite de 30M y Lidia y Jaime de 100k

```sql
ALTER USER ANA DEFAULT TABLESPACE VENTAS QUOTA 30M ON VENTAS;
ALTER USER EVA DEFAULT TABLESPACE VENTAS QUOTA 30M ON VENTAS;
ALTER USER LIDIA DEFAULT TABLESPACE PRODUCCION QUOTA 100K ON PRODUCCION;
ALTER USER JAIME DEFAULT TABLESPACE PRODUCCION QUOTA 100K ON PRODUCCION;
```

Y a pepe lo añadiremos con el tablespace system
```sql
ALTER USER PEPE DEFAULT TABLESPACE SYSTEM;
GRANT UNLIMITED TABLESPACE TO PEPE;
```

Crearemos un procedimiento 
```sql
Create or replace procedure DarPrivilegiosTablespaces(p_user varchar2)
is
begin
	dbms_output.put_line('Grant unlimited tablespace to '||p_user||';');
	dbms_output.put_line('Alter user '||p_user||' quota 0 on system'||';');
end;
/

Create or replace procedure ObjetosTablespaceProgramadores
is
	cursor c_users is
	Select GRANTEE
	from dba_role_privs
	where Granted_Role='PROGRAMADORES'
	and Grantee!='SYS';
begin
	for v_users in c_users loop
		DarPrivilegiosTablespaces(v_users.Grantee);
	end loop;
end;
/
```

Vamos a comprobar su funcionamiento
```sql
SQL> exec ObjetosTablespaceProgramadores();
Grant unlimited tablespace to JUAN;
Alter user JUAN quota 0 on system;
Grant unlimited tablespace to CLARA;
Alter user CLARA quota 0 on system;

Procedimiento PL/SQL terminado correctamente.
```

c) Pepe quiere crear una tabla Prueba que ocupe inicialmente 256K en el tablespace Ventas.

Nos conectamos al usuario Pepe y creamos la tabla con las indicaciones dadas.

```sql
Create table Prueba
( 
Prueba varchar2(2)
)
Tablespace Ventas
Storage
(
Initial 256K
);
```

d) Pepe decide que los programadores tengan acceso a la tabla Prueba antes creada y puedan ceder ese derecho y el de conectarse a la base de datos a los usuarios que ellos quieran.

```sql
GRANT READ ON PEPE.PRUEBA TO PROGRAMADORES WITH GRANT OPTION;
grant create session to programadores with admin option;
```



f) Lidia y Jaime dejan la empresa, borra los usuarios y el espacio de tablas correspondiente, detalla los pasos necesarios para que no quede rastro del espacio de tablas.

```sql
drop user Lidia;
drop user Jaime;
drop tablespace Produccion including contents and datafiles;
```

