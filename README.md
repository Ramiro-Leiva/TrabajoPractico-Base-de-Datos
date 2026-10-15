# Base de datos: TPBaseDatos

## Descripción

Este proyecto consiste en el diseño e implementación de una base de datos relacional para organizar y consultar información sobre el intercambio comercial, incluyendo exportaciones e importaciones, países, zonas comerciales, categorías económicas y rubros de uso económico.

El objetivo es organizar los datos en tablas relacionadas y facilitar su consulta mediante sentencias SQL y vistas.

Los datos fueron extraídos desde: [indec.gob.ar](https://www.indec.gob.ar/ftp/ica_digital/ica_d_08_26E158B1D119/#Principales%20socios) 

## Tecnologías utilizadas

- **Sistema gestor de base de datos:** MariaDB.
- **Herramienta de administración:** HeidiSQL.
- **Lenguaje de consultas:** SQL.

## Estructura de la base de datos

La base de datos contiene las siguientes tablas:

- `datos`: tabla utilizada para almacenar los datos de origen.
```sql
CREATE TABLE datos(id INT NOT NULL PRIMARY KEY AUTO_INCREMENT, d_anio INT(4) UNSIGNED NOT NULL,
 d_mes INT(2) UNSIGNED ZEROFILL NOT NULL, d_tipocomercio VARCHAR(50) NOT NULL,
d_zonacomercial VARCHAR(50) NOT NULL, d_pais VARCHAR(50) NOT NULL,
d_rubrousoecnomico VARCHAR(100) NOT NULL, d_categoriaeconomica VARCHAR(100) NOT NULL, d_valor DECIMAL (18,2) NOT NULL);
```
- `tipocomercio`: identifica el tipo de comercio, como exportaciones o importaciones.
 ```sql
-- Tipo Comercio:
CREATE TABLE tipocomercio(tc_codi SMALLINT NOT NULL PRIMARY KEY AUTO_INCREMENT,
tc_descrip VARCHAR(20) UNIQUE NOT NULL);
```
- `zonacomercial`: organiza los países por zona comercial.
 ```sql
-- Zona Comercial:
CREATE TABLE zonacomercial(zc_codi SMALLINT NOT NULL PRIMARY KEY AUTO_INCREMENT,
zc_nombre VARCHAR(50) UNIQUE NOT NULL);
```
- `rubrousoeconomico`: contiene los rubros de uso económico.
```sql
-- Rubro Uso Economico
CREATE TABLE rubrousoeconomico(rue_codi SMALLINT NOT NULL PRIMARY KEY AUTO_INCREMENT,
rue_descrip VARCHAR(100) UNIQUE NOT NULL);
```
- `pais`: contiene los países.
 ```sql
-- Pais:
CREATE TABLE pais(p_codi SMALLINT NOT NULL PRIMARY KEY AUTO_INCREMENT, p_nombre VARCHAR(50) NOT NULL,
p_zonac SMALLINT NOT NULL, UNIQUE (p_nombre, p_zonac), FOREIGN KEY (p_zonac) REFERENCES zonacomercial(zc_codi));
```
- `categoriaeconomica`: almacena las categorías económicas.
 ```sql
-- CategoriaEconomica:
CREATE TABLE categoriaeconomica(ce_codi SMALLINT NOT NULL PRIMARY KEY AUTO_INCREMENT, ce_descrip VARCHAR(100) NOT NULL,
ce_rue SMALLINT NOT NULL, UNIQUE (ce_descrip, ce_rue), FOREIGN KEY(ce_rue) REFERENCES rubrousoeconomico(rue_codi));
```
- `intercambiocomercial`: almacena los registros de intercambio comercial.
 ```sql
-- Intercambios Comerciales
CREATE TABLE intercambiocomercial(ic_codi INT NOT NULL PRIMARY KEY AUTO_INCREMENT, ic_anio SMALLINT(4) UNSIGNED NOT NULL,
ic_mes SMALLINT(2) UNSIGNED ZEROFILL NOT NULL, ic_valor DECIMAL(18,2) UNSIGNED NOT NULL, ic_tipo SMALLINT NOT NULL,
ic_pais SMALLINT NOT NULL, ic_categoriae SMALLINT NOT NULL, FOREIGN KEY(ic_tipo) REFERENCES tipocomercio(tc_codi),
FOREIGN KEY (ic_pais) REFERENCES pais (p_codi), FOREIGN KEY (ic_categoriae) REFERENCES categoriaeconomica(ce_codi));
```

Las tablas se relacionan mediante claves primarias y claves foráneas, según corresponda.

## Vistas

El proyecto incluye las siguientes vistas:

- `vista_comercio_por_pais`
 ```sql
CREATE VIEW vista_comercio_por_pais AS
SELECT p.p_nombre AS pais, tc.tc_descrip AS tipo_comercio, ce.ce_descrip AS categoria_economica,
rue.rue_descrip AS rubro, ic.ic_anio AS anio, ic.ic_mes AS mes, SUM(ic.ic_valor) AS valor_total
FROM intercambiocomercial ic, pais p, tipocomercio tc, categoriaeconomica ce, rubrousoeconomico rue
WHERE ic.ic_pais = p.p_codi AND ic.ic_tipo = tc.tc_codi AND  ic.ic_categoriae = ce.ce_codi AND
ce.ce_rue = rue.rue_codi
GROUP BY pais, tipo_comercio, categoria_economica, rubro,anio, mes;
```
Comprobamos su funcionamiento:
```sql
SELECT pais, tipo_comercio, categoria_economica, rubro, anio, mes ,valor_total
FROM vista_comercio_por_pais
WHERE pais = 'Brasil' AND tipo_comercio = 'Importaciones'
GROUP BY anio, mes, tipo_comercio, categoria_economica;
``` 
- `vista_intercambio_detallado`
 ```sql
CREATE VIEW vista_intercambio_detallado AS
SELECT ic.ic_codi AS codigo, ic.ic_anio AS anio, ic.ic_mes AS mes, tc.tc_descrip AS tipo_comercio,
p.p_nombre AS pais, zc.zc_nombre AS zona_comercial, rue.rue_descrip AS rubro_uso_economico,
ce.ce_descrip AS categoria_economica, ic.ic_valor AS valor
FROM intercambiocomercial ic, tipocomercio tc, pais p, zonacomercial zc, categoriaeconomica ce, rubrousoeconomico rue
WHERE ic.ic_tipo = tc.tc_codi AND ic.ic_pais = p.p_codi AND p.p_zonac = zc.zc_codi AND
 ic.ic_categoriae = ce.ce_codi AND ce.ce_rue = rue.rue_codi;
```
Comprobamos su funcionamiento: 
```sql
SELECT * FROM vista_intercambio_detallado;
```
- `vista_totales_por_categoria`
```sql
CREATE VIEW vista_totales_por_categoria AS
SELECT ic.ic_anio AS anio, ic.ic_mes AS mes, tc.tc_descrip AS tipo_comercio, rue.rue_descrip AS rubro_uso_economico,
ce.ce_descrip AS categoria_economica, SUM(ic.ic_valor) AS total_valor
FROM intercambiocomercial ic, tipocomercio tc, categoriaeconomica ce, rubrousoeconomico rue
WHERE ic.ic_tipo = tc.tc_codi AND ic.ic_categoriae = ce.ce_codi AND ce.ce_rue = rue.rue_codi
GROUP BY anio, mes, tipo_comercio, rubro_uso_economico, categoria_economica;
```
Comprobamos su funcionamiento:
```sql
SELECT * FROM vista_totales_por_categoria
ORDER BY anio, mes, total_valor DESC;
```
- `vista_totales_por_pais`
```sql
CREATE VIEW vista_totales_por_pais AS
SELECT ic.ic_anio AS anio, ic.ic_mes AS mes, tc.tc_descrip AS tipo_comercio, p.p_nombre AS pais, SUM(ic.ic_valor) AS total_valor
FROM intercambiocomercial ic, tipocomercio tc, pais p
WHERE ic.ic_tipo = tc.tc_codi AND ic.ic_pais = p.p_codi
GROUP BY ic.ic_anio, ic.ic_mes, tc.tc_descrip, p.p_codi, p.p_nombre;
```
Comprobamos su funcionamiento:
```sql
SELECT * FROM vista_totales_por_pais
ORDER BY pais, anio, tipo_comercio;
```
Estas vistas permiten consultar información del intercambio comercial, obtener detalles de los registros y agrupar valores por país o categoría económica.

## Gráfico DER.

<img width="889" height="804" alt="TrabajoPracticoBD" src="https://github.com/user-attachments/assets/48ac371c-14f8-494a-b3a0-c4d2c4e557d2"/>

## Instalación / Recreación de la Base de Batos
- Recrear:
  
1. Descargar el CSV con los datos: 
2. Abrir HeidiSQL y conectarse al servidor de base de datos.
3. Crear una nueva base de datos.
4. Copiar y ejecutar la sentencia SQL de `datos` desde la [Estructura de la base de datos](#estructura-de-la-base-de-datos).
6. Ir a Herramientas, Importar Archivo CSV...
   Seleccionar el archivo descargado, seleccionar la base de datos y la tabla en la cual serán cargados los datos, deseleccionar el          campo `id` e importar.
   <img width="650" height="639" alt="image" src="https://github.com/user-attachments/assets/974ad8a7-c333-47d1-9c3e-39b76b3f396f"/>
7. Copiar y ejecutar en orden las sentencias SQL desde la [Estructura de la base de datos](#estructura-de-la-base-de-datos)
8. Una vez creadas las tablas, cargar los datos dentro de las tablas correspondientes:
 ```sql
-- Carga de datos: 
-- ===========================================
-- Cargar Tipos de Comercio:
INSERT INTO tipocomercio (tc_descrip)
SELECT DISTINCT d.d_tipocomercio
FROM datos d 
WHERE d.d_tipocomercio IS NOT NULL;

-- Cargar Zonas Comerciales:
INSERT INTO zonacomercial (zc_nombre)
SELECT DISTINCT d.d_zonacomercial 
FROM datos d
WHERE d.d_zonacomercial IS NOT NULL;

-- Cargar Rubros / Usos Economicos:
INSERT INTO rubrousoeconomico (rue_descrip)
SELECT DISTINCT d.d_rubrousoeconomico
FROM datos d
WHERE d.d_rubrousoeconomico IS NOT NULL;

-- Cargar Paises:
INSERT INTO pais (p_nombre, p_zonac)
SELECT DISTINCT d.d_pais, z.zc_codi
FROM datos d
JOIN zonacomercial z ON z.zc_nombre = d.d_zonacomercial;

-- Cargar Categorias Economicas:
INSERT INTO categoriaeconomica (ce_descrip, ce_rue)
SELECT DISTINCT d.d_categoriaeconomica, r.rue_codi
FROM datos d
JOIN rubrousoeconomico r ON r.rue_descrip = d.d_rubrousoeconomico;

-- Cargar Intercambios Comerciales:
INSERT INTO intercambiocomercial (ic_anio, ic_mes, ic_valor, ic_tipo, ic_pais, ic_categoriae)
SELECT d.d_anio, d.d_mes, d.d_valor, tc.tc_codi, p.p_codi, c.ce_codi
FROM datos d
JOIN tipocomercio tc ON tc.tc_descrip = d.d_tipocomercio
JOIN zonacomercial z ON z.zc_nombre = d.d_zonacomercial
JOIN pais p ON p.p_nombre = d.d_pais AND p.p_zonac = z.zc_codi
JOIN rubrousoeconomico r ON r.rue_descrip = d.d_rubrousoeconomico
JOIN categoriaeconomica c ON c.ce_descrip = d.d_categoriaeconomica AND c.ce_rue = r.rue_codi;
```
8. Copiar y ejecutar las sentencias SQL de las [Vistas](#vistas).

- Instalar: 
1. Descargar el archivo [`BackUpDataBase.sql`](https://github.com/Ramiro-Leiva/TrabajoPractico-Base-de-Datos/blob/main/BackUpDataBase.sql)
2. Abrir HeidiSQL y conectarse al servidor de base de datos.
3. Crear o seleccionar la base de datos `tpbasedatos`.
4. Importar el archivo SQL del proyecto.
5. Ejecutar las consultas para comprobar que las tablas, relaciones y vistas se hayan creado correctamente.

## Archivos del proyecto

- `BackUpDataBase.sql`: script SQL con la estructura y los objetos de la base de datos. [Ver Archivo...](https://github.com/Ramiro-Leiva/TrabajoPractico-Base-de-Datos/blob/main/BackUpDataBase.sql)
- `Sentencias-DDL.pdf`: documentación de tablas, campos, índices y relaciones. [Ver Archivo...](https://github.com/Ramiro-Leiva/TrabajoPractico-Base-de-Datos/blob/main/Documentaci%C3%B3n%20-%20Secuencias%20DDL.pdf)
- `TrabajoPracticoBD.png`: Diagrama Entidad Relación. [Ver Archivo...](https://github.com/Ramiro-Leiva/TrabajoPractico-Base-de-Datos/blob/main/TrabajoPracticoBD.png) 
## Autores

- Ramiro **LEIVA**.
- Fernando **TORRES**.

