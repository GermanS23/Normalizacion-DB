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

1. Seleccionar el dataset que queramos normalizar, en formato csv o xlsx, en este caso se utilizará:

https://www.kaggle.com/datasets/orkunaktas/all-football-players-stats-in-top-5-leagues-2324


2. Dirigirnos a nuestro lugar de trabajo, puede ser Headi o MySQLWorkbench, donde crearemos y utilizaremos una nueva base de datos, realizando:
```sql
    create database ligas;
```
```sql
    use ligas;
```

3. Crear la tabla  principal donde cargaremos todos los datos de nuestro archivo csv

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
4. Cargar los datos del archivo csv
```sql
LOAD DATA INFILE 'C:\\ligas\\Gestion-BaseDeDatos.csv'
INTO TABLE jugadores 
FIELDS TERMINATED BY ';' 
LINES TERMINATED BY '\n' 
IGNORE 1 ROWS;
```
5. Crear las entidades que tendrá nuestra base de datos, partiendo de la tabla principal *jugadores* e inserción de datos en las tablas 

*Posicion*
```sql
CREATE TABLE posicion(
pos_cod int not null auto_increment primary key,
pos_descrip varchar(255) not null
);
INSERT INTO posicion (pos_descrip)
SELECT DISTINCT
   	SUBSTRING_INDEX(jug_posicion, ',', -1)
FROM jugadores;
```
*Equipos*
```sql
CREATE TABLE equipos(
equip_cod int not null auto_increment primary key,
equip_nombre varchar(255) not null

INSERT INTO equipos (equip_nombre)
SELECT DISTINCT jug_equipo FROM jugadores;
);
```
*Competicion*
```sql
CREATE TABLE competicion(
comp_cod int not null auto_increment primary key,
comp_nombre varchar(255) not null
);
INSERT INTO competicion (comp_nombre)
SELECT DISTINCT
   	SUBSTRING_INDEX(jug_competicion, ' ', -2)
FROM jugadores;
```
*Nacionalidad*
```sql
CREATE TABLE nacionalidad(
nac_cod int not null auto_increment primary KEY,
nac_abrev VARCHAR(255) NOT NULL,
nac_nombre varchar(255) not null
);
INSERT INTO nacionalidad (nac_abrev, nac_nombre)
SELECT DISTINCT 
    SUBSTRING(jug_nacionalidad, 1, LOCATE(' ', jug_nacionalidad) - 1) AS abreviatura,
    SUBSTRING(jug_nacionalidad, LOCATE(' ', jug_nacionalidad) + 1) AS nombre
FROM jugadores;
```

*Tabla intermedio para la relación de muchos a muchos entre Jugadores y Posicion*
```sql
    CREATE TABLE jugador_posicion (
    jp_jug_cod INT NOT NULL,
    jp_pos_cod INT NOT NULL,
    PRIMARY KEY (jp_jug_cod, jp_pos_cod),
    FOREIGN KEY (jp_jug_cod) REFERENCES jugadores(jug_cod),
    FOREIGN KEY (jp_pos_cod) REFERENCES posicion(pos_cod)
);
```

*Carga de datos a la tabla intermedio, con el objetivo de que se carguen los codigos del jugador y las distintas posiciones que tenga"
```sql
    INSERT INTO jugador_posicion (jp_jug_cod, jp_pos_cod)
        SELECT 
            jug_cod,
                (
                    SELECT pos_cod FROM posicion WHERE pos_descrip = SUBSTRING_INDEX(jug_posicion, ',', 1)
                )
        FROM 
            jugadores
        WHERE 
            jug_posicion LIKE '%,%'
        UNION ALL
        SELECT 
            jug_cod,
                (
                    SELECT pos_cod FROM posicion WHERE pos_descrip = SUBSTRING_INDEX(jug_posicion, ',', -1)
                )
        FROM 
            jugadores
        WHERE 
            jug_posicion LIKE '%,%'
        UNION ALL
        SELECT 
            jug_cod,
                (
                    SELECT pos_cod FROM posicion WHERE pos_descrip = jug_posicion
                )
        FROM 
            jugadores
        WHERE 
            jug_posicion NOT LIKE '%,%';
```
6. Creación y Modificación de la columna existente, de la tabla principal jugadores.
  -  Creación de una tabla temporal para almacenar los datos de las posiciones de los jugadores, sacandolo de la tabla intermedio.
  
      ```sql
      ALTER TABLE jugadores ADD COLUMN jug_pos_temp VARCHAR(255) after jug_nacionalidad;
      ```
  - Carga de datos a la tabla temporal
       ```sql
           UPDATE jugadores
            SET jug_pos_temp = (
                SELECT GROUP_CONCAT(jp_pos_cod SEPARATOR ',') 
                FROM jugador_posicion
                WHERE jp_jug_cod = jugadores.jug_cod
            );
       ```
  - Eliminamos la tabla original "jug_posicion" y transformamos la temporal en la nueva original.

      ```sql
         ALTER TABLE jugadores DROP COLUMN jug_posicion
      ```
      ```sql
        ALTER TABLE jugadores CHANGE jug_pos_temp jug_posicion VARCHAR(255)
      ```
      
7. Se repite el mismo procedimiento para las tablas, nacionalidad, equipos y competición

    *Nacionalidad*
   
    ```sql
       ALTER TABLE jugadores ADD COLUMN jug_nac_temp INT  NOT NULL AFTER jug_nombre;
    ```

   ```sql
       UPDATE jugadores SET jug_nac_temp = (SELECT nac_cod FROM nacionalidad WHERE nac_abrev = SUBSTRING(jug_nacionalidad, 1,
       LOCATE(' ', jug_nacionalidad)-1)) WHERE jug_cod > 0;
   ```
   
   ```sql
       ALTER TABLE jugadores DROP COLUMN jug_nacionalidad;
   ```
   
   ```sql
       ALTER TABLE jugadores CHANGE jug_nac_temp jug_nacionalidad INT NOT NULL;
   ```

   ```sql
       ALTER TABLE jugadores ADD CONSTRAINT fk_jugadores_nacionalidad FOREIGN KEY (jug_nacionalidad) REFERENCES nacionalidad(nac_cod);
   ```

   *Equipos*

   ```sql
       ALTER TABLE jugadores ADD COLUMN jug_equipos_temp INT AFTER jug_posicion;
   ```
   
   ```sql
       UPDATE jugadores SET jug_equipos_temp = (SELECT equip_cod FROM equipos WHERE equip_nombre = jugadores.jug_equipo)
       WHERE jug_cod > 0;
   ```
   
   ```sql
       ALTER TABLE jugadores DROP COLUMN jug_equipo;
   ```
   
   ```sql
       ALTER TABLE jugadores CHANGE jug_equipos_temp jug_equipo INT;
   ```
   
   ```sql
       ALTER TABLE jugadores ADD CONSTRAINT fk_jugadores_equipo FOREIGN KEY (jug_equipo) REFERENCES equipos(equip_cod);
   ```

   *Competición*

   ```sql
       ALTER TABLE jugadores ADD COLUMN jug_competicion_temp INT NOT NULL AFTER jug_equipo;
   ```
   
   ```sql
       UPDATE jugadores SET jug_competicion_temp = ( SELECT comp_cod FROM competicion
       WHERE comp_nombre = SUBSTRING_INDEX(jug_competicion, '',-2)where jug_cod > 0;
   ```
   
   ```sql
       ALTER TABLE jugadores DROP COLUMN jug_competicion;
   ```
   
   ```sql
       ALTER TABLE jugadores CHANGE jug_competicion_temp jug_competicion INT;
   ```

   ```sql
       ALTER TABLE jugadores ADD CONSTRAINT fk_jugadores_competicion FOREIGN KEY (jug_competicion) REFERENCES competicion(comp_cod);
   ```


8. Cambio de los tipos de datos de la tabla principal "jugadores"   
 
    
```sql
       ALTER TABLE jugadores
MODIFY jug_mj TINYINT,
MODIFY jug_str TINYINT,
MODIFY jug_gls TINYINT,
MODIFY jug_ast TINYINT,
MODIFY jug_gls_ast TINYINT,
MODIFY jug_gls_sinpenales TINYINT,
MODIFY jug_gl_penales TINYINT,
MODIFY jug_pen_int_jug TINYINT,
MODIFY jug_amarillas TINYINT,
MODIFY jug_rojas TINYINT,
MODIFY jug_gol_esp_sinpen DECIMAL,
MODIFY jug_ast_esp DECIMAL,
MODIFY jug_gol_esp_ast_esp_sinpen DECIMAL,
MODIFY jug_PrgC DECIMAL,
MODIFY jug_xG DECIMAL,
MODIFY jug_xAG DECIMAL,
MODIFY jug_xGxAG DECIMAL,
MODIFY jug_npxG DECIMAL,
MODIFY jug_npxGxAG DECIMAL,
MODIFY jug_90s INT,
MODIFY jug_gol_esp INT,
MODIFY jug_PrgP INT,
MODIFY jug_PrgR INT,
MODIFY jug_GAPK INT,
MODIFY jug_nacimiento YEAR,
MODIFY jug_edad TINYINT ;
```
   
   
9. Consultas
  - Esta consulta combina las tablas jugadores, jugador_posicion y posicion para mostrar el código del jugador, el nombre del jugador y la lista de posiciones, ordenados por jug_cod.
       ```sql 
            SELECT 
                j.jug_cod, 
                j.jug_nombre, 
                GROUP_CONCAT(DISTINCT p.pos_descrip ORDER BY p.pos_descrip SEPARATOR ', ') AS posiciones
            FROM 
                jugadores j
            JOIN 
                jugador_posicion jp ON j.jug_cod = jp.jp_jug_cod
            JOIN 
                posicion p ON jp.jp_pos_cod = p.pos_cod
            GROUP BY 
                j.jug_cod, j.jug_nombre
            ORDER BY
                j.jug_cod;
      ```

