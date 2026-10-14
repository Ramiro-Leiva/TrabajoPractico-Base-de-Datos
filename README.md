# Base de datos: TPBaseDatos

## Descripción

Este proyecto consiste en el diseño e implementación de una base de datos relacional para organizar y consultar información sobre el intercambio comercial, incluyendo exportaciones e importaciones, países, zonas comerciales, categorías económicas y rubros de uso económico.

El objetivo es organizar los datos en tablas relacionadas y facilitar su consulta mediante sentencias SQL y vistas.

## Tecnologías utilizadas

- **Sistema gestor de base de datos:** MariaDB.
- **Herramienta de administración:** HeidiSQL.
- **Lenguaje de consultas:** SQL.

## Estructura de la base de datos

La base de datos contiene las siguientes tablas:

- `intercambiocomercial`: almacena los registros de intercambio comercial.
 ```sql
-- Intercambios Comerciales
CREATE TABLE intercambiocomercial(ic_codi INT NOT NULL PRIMARY KEY AUTO_INCREMENT, ic_anio SMALLINT(4) UNSIGNED NOT NULL,
ic_mes SMALLINT(2) UNSIGNED ZEROFILL NOT NULL, ic_valor DECIMAL(18,2) UNSIGNED NOT NULL, ic_tipo SMALLINT NOT NULL,
ic_pais SMALLINT NOT NULL, ic_categoriae SMALLINT NOT NULL, FOREIGN KEY(ic_tipo) REFERENCES tipocomercio(tc_codi),
FOREIGN KEY (ic_pais) REFERENCES pais (p_codi), FOREIGN KEY (ic_categoriae) REFERENCES categoriaeconomica(ce_codi));
```
- `pais`: contiene los países.
 ```sql
-- Pais:
CREATE TABLE pais(p_codi SMALLINT NOT NULL PRIMARY KEY AUTO_INCREMENT, p_nombre VARCHAR(50) NOT NULL,
p_zonac SMALLINT NOT NULL, UNIQUE (p_nombre, p_zonac), FOREIGN KEY (p_zonac) REFERENCES zonacomercial(zc_codi));
```
- `zonacomercial`: organiza los países por zona comercial.
 ```sql
-- Zona Comercial:
CREATE TABLE zonacomercial(zc_codi SMALLINT NOT NULL PRIMARY KEY AUTO_INCREMENT,
zc_nombre VARCHAR(50) UNIQUE NOT NULL);
```
- `tipocomercio`: identifica el tipo de comercio, como exportaciones o importaciones.
 ```sql
-- Tipo Comercio:
CREATE TABLE tipocomercio(tc_codi SMALLINT NOT NULL PRIMARY KEY AUTO_INCREMENT,
tc_descrip VARCHAR(20) UNIQUE NOT NULL);
```
- `categoriaeconomica`: almacena las categorías económicas.
 ```sql
-- CategoriaEconomica:
CREATE TABLE categoriaeconomica(ce_codi SMALLINT NOT NULL PRIMARY KEY AUTO_INCREMENT, ce_descrip VARCHAR(100) NOT NULL,
ce_rue SMALLINT NOT NULL, UNIQUE (ce_descrip, ce_rue), FOREIGN KEY(ce_rue) REFERENCES rubrousoeconomico(rue_codi));
```
- `rubrousoeconomico`: contiene los rubros de uso económico.
```sql
-- Rubro Uso Economico
CREATE TABLE rubrousoeconomico(rue_codi SMALLINT NOT NULL PRIMARY KEY AUTO_INCREMENT,
rue_descrip VARCHAR(100) UNIQUE NOT NULL);
```
- `datos`: tabla utilizada para almacenar los datos de origen.

Las tablas se relacionan mediante claves primarias y claves foráneas, según corresponda.

## Vistas

El proyecto incluye las siguientes vistas:

- `vista_comercio_por_pais`
- `vista_intercambio_detallado`
- `vista_totales_por_categoria`
- `vista_totales_por_pais`

Estas vistas permiten consultar información del intercambio comercial, obtener detalles de los registros y agrupar valores por país o categoría económica.

## Gráfico DER.

<img width="889" height="804" alt="TrabajoPracticoBD" src="https://github.com/user-attachments/assets/48ac371c-14f8-494a-b3a0-c4d2c4e557d2" />

## Instalación

1. Instalar MariaDB y HeidiSQL.
2. Abrir HeidiSQL y conectarse al servidor de base de datos.
3. Crear o seleccionar la base de datos `tpbasedatos`.
4. Importar el archivo SQL del proyecto.
5. Ejecutar las consultas para comprobar que las tablas, relaciones y vistas se hayan creado correctamente.

## Archivos del proyecto

- `TPDB.sql`: script SQL con la estructura y los objetos de la base de datos.
- `Sentencias-DDL.pdf`: documentación de tablas, campos, índices y relaciones.

## Autores

- Ramiro **LEIVA**.
- Fernando **TORRES**.

