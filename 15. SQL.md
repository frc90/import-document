# SQL, MySQL, PostgreSQL

**Índice**

1. [SQL](#id1)

# 1. SQL

<div id='id1' />

**SQL**

Este comando conectara al servidor de **MySQL** que tengas puesto

```batch
> mysql -h 127.0.0.1 -u root -p
```

_nota:_ Esto es por consola.

```batch
mysql> select version(), current_date;
```

+-----------+--------------+
| version() | current_date |
+-----------+--------------+
| 8.0.30 | 2025-02-12 |
+-----------+--------------+

## En el apartado _Query_

- Crear base de datos

```sql
CREATE DATABASE zoo;
```

- Mostrar bases de datos

```sql
SHOW DATABASES;
```

- Mostrar las tablas de la base de datos

```sql
SHOW TABLES;
```

- Crear una tabla

```sql
CREATE TABLE mascota(
nombre VARCHAR(20),
propietario VARCHAR(20),
especie VARCHAR(20),
sexo CHAR(1),
nacimiento DATE,
fallecimiento DATE);
```

- Cargar datos de **archivo.txt** localmente, se necesita hacer una serie de pasos

```batch
mysql> SET GLOBAL local_infile=1;
```

- Cerrar el servidor y volverlo abrir

```batch
mysql> quit
```

```batch
mysql --local-infile=1 -u root -p1
```

```batch
mysql> SET GLOBAL local_infile=1;
```
