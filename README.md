# Normalizacion-DB

## Introducción
El presente repositorio contiene la documentación necesaria para la normalización de una base de datos a partir de un archivo csv.

### Base de Datos de estadistica sobre los jugadores de las 5 Grandes Ligas del Mundo.

Esta base de datos contiene toda la información sobre todas las estadisticas de la temporada 23/24 de las 5 mejores ligas de fútbol.

## Instalación

1- Clona este repositorio
```bash
    git clone https://github.com/GermanS23/Normalizacion-DB.git
 ```
## Uso

1- 


## Paso a Paso para la normalización de un archivo csv

1- Seleccionar el dataset que queramos normalizar, en formato csv o xlsx, en este caso se utilizará:

https://www.kaggle.com/datasets/orkunaktas/all-football-players-stats-in-top-5-leagues-2324


3- Dirigirnos a nuestro lugar de trabajo, puede ser Headi o MySQLWorkbench, donde crearemos y utilizaremos una nueva base de datos, realizando:
```sql
    create database ligas;
```
```sql
    use ligas;
```

4- Crear la tabla  principal donde cargaremos todos los datos de nuestro archivo csv

```sql
    CREATE TABLE jugadores (
 jug_cod INT NOT NULL PRIMARY KEY,
 jug_nombre VARCHAR(255) NOT NULL,
 jug_nacionalidad VARCHAR(255) NOT NULL,
 jug_posicion VARCHAR(255) NOT NULL,
 jug_equipo VARCHAR(255) NOT NULL,
 jug_competicion VARCHAR(255) NOT NULL,
 jug_edad VARCHAR(255) NOT NULL,
 jug_nacimiento VARCHAR(255) NOT NULL,
 jug_mj VARCHAR(255) NOT NULL,
 jug_str VARCHAR(255) NOT NULL,
 jug_90s VARCHAR(255) NOT NULL,
 jug_gls VARCHAR(255) NOT NULL,
 jug_ast VARCHAR(255) NOT NULL,
 jug_gls_ast VARCHAR(255) NOT NULL,
 jug_gls_sinpenales VARCHAR(255) NOT NULL,
 jug_gl_penales VARCHAR(255) NOT NULL,
 jug_pen_int_jug VARCHAR(255) NOT NULL,
 jug_amarillas VARCHAR(255) NOT NULL,
 jug_rojas VARCHAR(255) NOT NULL,
 jug_gol_esp VARCHAR(255) NOT NULL,
 jug_gol_esp_sinpen VARCHAR(255) NOT NULL,
 jug_ast_esp VARCHAR(255) NOT NULL,
 jug_gol_esp_ast_esp_sinpen VARCHAR(255) NOT NULL,
 jug_PrgC VARCHAR(255)NOT NULL,
 jug_PrgP VARCHAR(255) NOT NULL,
 jug_PrgR VARCHAR(255) NOT NULL,
 jug_GAPK VARCHAR(255) NOT NULL,
 jug_xG VARCHAR(255) NOT NULL,
 jug_xAG VARCHAR(255) NOT NULL,
 jug_xGxAG VARCHAR(255) NOT NULL,
 jug_npxG VARCHAR(255) NOT NULL,
 jug_npxGxAG VARCHAR(255) NOT NULL 
 );

```
5- Cargar los datos del archivo csv
```sql
LOAD DATA INFILE 'C:\\ligas\\Gestion-BaseDeDatos.csv'
INTO TABLE jugadores 
FIELDS TERMINATED BY ';' 
LINES TERMINATED BY '\n' 
IGNORE 1 ROWS;
```
6- Crear las entidades que tendrá nuestra base de datos, partiendo de la tabla principal *jugadores* e inserción de datos 

*Posicion*
```sql
create table posicion(
pos_cod int not null auto_increment primary key,
pos_descrip varchar(255) not null
);
INSERT INTO posicion (pos_nom)
SELECT DISTINCT jug_Pos FROM jugadores;
```

