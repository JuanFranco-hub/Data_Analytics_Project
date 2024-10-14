# Crear y Gestionar una Base de Datos en Azure: Relación, Normalización y Optimización de Consultas

## Índice

1. [Introducción](#introducción)
2. [Requisitos Previos](#requisitos-previos)
3. [Paso 1: Crear una Base de Datos en Azure](#paso-1-crear-una-base-de-datos-en-azure)
4. [Paso 2: Crear y Guardar Tablas](#paso-2-crear-y-guardar-tablas)
5. [Paso 3: Relacionar Tablas](#paso-3-relacionar-tablas)
6. [Paso 4: Optimización de Consultas](#paso-4-optimización-de-consultas)

## Introducción

Este tutorial te guiará paso a paso sobre cómo:

1. Crear una base de datos en Azure SQL Database.
2. Crear tablas y almacenarlas en dicha base de datos, utilizando el esquema de las tablas `customers`, `orders`, `order_status`, y `geo_lookup`.
3. Relacionar las tablas mediante llaves foráneas.
4. Normalizar las tablas para asegurar la eficiencia y evitar redundancias.
5. Optimizar las consultas para mejorar el rendimiento.

## Requisitos Previos

- Cuenta de Azure activa.
- Conocimientos básicos de SQL.
- Azure SQL Database configurada (puedes seguir este [enlace](https://docs.microsoft.com/en-us/azure/azure-sql/database/sql-database-get-started) para aprender cómo crear una).

## Paso 1: Crear una Base de Datos en Azure

### 1.1 Acceder a Azure

- Inicia sesión en [Azure Portal](https://portal.azure.com/).
- Navega a "SQL Databases" y selecciona "Create".

### 1.2 Configuración de la Base de Datos

- **Subscription**: Selecciona tu suscripción.
- **Resource Group**: Crea uno o selecciona uno existente.
- **Database Name**: Elige un nombre para tu base de datos, por ejemplo, `ElistElectronicsDB`.
- **Server**: Crea un nuevo servidor o selecciona uno existente. Asegúrate de que el servidor sea accesible desde tu IP local.
- **Performance**: Elige un plan de rendimiento adecuado a tus necesidades (Básico para pruebas, Standard para producción).

Una vez configurada, haz clic en "Review + Create" y luego en "Create". Espera a que se despliegue.

### 1.3 Conectar a la Base de Datos

Puedes usar herramientas como [Azure Data Studio](https://docs.microsoft.com/en-us/sql/azure-data-studio/download?view=sql-server-ver15) o [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) para conectarte a la base de datos. La cadena de conexión se encuentra en la sección "Connection Strings" del servidor en Azure Portal.

### 2.1 Crear Tablas en SQL

Usa los siguientes scripts para crear las tablas:

```sql
-- Crear la tabla geo_lookup
CREATE TABLE geo_lookup (
    country NVARCHAR(100) UNIQUE,
    region NVARCHAR(255)
);

-- Crear la tabla customers
CREATE TABLE customers (
    id NVARCHAR(100) PRIMARY KEY,
    marketing_channel NVARCHAR(100),
    account_creation_method NVARCHAR(100),
    country_code NVARCHAR(100),
    loyalty_program TINYINT,
    created_on DATETIME
);

-- Crear la tabla order_status
CREATE TABLE order_status (
    id NVARCHAR(100) PRIMARY KEY,
    purchase_ts DATETIME,
    ship_ts DATETIME,
    delivery_ts DATETIME,
    refund_ts DATETIME,
    order_id NVARCHAR(100) -- Clave foránea de id en orders
);

-- Crear la tabla orders
CREATE TABLE orders (
    id NVARCHAR(100) PRIMARY KEY,
    customer_id NVARCHAR(100),
    purchase_ts DATETIME,
    product_id NVARCHAR(100),
    product_name NVARCHAR(255),
    currency NVARCHAR(10),
    local_price DECIMAL(10, 2),
    usd_price DECIMAL(10, 2),
    purchase_platform NVARCHAR(100)
);

SELECT name FROM sys.tables;
```

---

### **2.2 Insertar Datos Usando Python y SQLAlchemy**

Para insertar datos en las tablas de la base de datos en Azure SQL a partir de archivos CSV, utilizamos Python con las bibliotecas **SQLAlchemy** y **Pandas**. El código proporciona una forma eficiente de conectarse a la base de datos, leer los archivos CSV desde este repositorio y cargarlos en las tablas correspondientes.

El proceso se desglosa en los siguientes pasos:

#### 2.2.1 Configuración de la Conexión a la Base de Datos

```python
config = {
    'db': {
        'server': 'your_server_name.database.windows.net',
        'database': 'ElistElectronicsDB',
        'username': 'your_username',
        'password': 'your_password',
        'driver': 'ODBC Driver 18 for SQL Server'
    },
    'csv_urls': {
        'customers': 'https://raw.githubusercontent.com/JuanFranco-hub/Data_Analytics_Project/refs/heads/main/Datos/customers.csv',
        'geo_lookup': 'https://raw.githubusercontent.com/JuanFranco-hub/Data_Analytics_Project/refs/heads/main/Datos/geo_lookup.csv',
        'order_status': 'https://raw.githubusercontent.com/JuanFranco-hub/Data_Analytics_Project/refs/heads/main/Datos/order_status.csv',
        'orders': 'https://raw.githubusercontent.com/JuanFranco-hub/Data_Analytics_Project/refs/heads/main/Datos/orders.csv'
    }
}
```

#### 2.2.2 Creación de la Cadena de Conexión y el Motor SQLAlchemy

```python
# Crear la cadena de conexión usando el diccionario
connection_string = f"mssql+pyodbc://{config['db']['username']}:{config['db']['password']}@" \
                    f"{config['db']['server']}/{config['db']['database']}?driver={config['db']['driver']}"

# Crear el motor de conexión con SQLAlchemy
engine = create_engine(connection_string)
```

#### 2.2.3 Función para Cargar CSV en la Base de Datos

```python
def cargar_csv_a_sql(csv_url, tabla_sql):
    # Leer los datos del CSV desde GitHub
    df = pd.read_csv(csv_url)
    
    # Cargar los datos en la tabla SQL (si la tabla ya existe, se agregan los datos)
    df.to_sql(tabla_sql, con=engine, if_exists='append', index=False)
    print(f"Datos cargados en la tabla: {tabla_sql}")
```

Esta función es la encargada de cargar los datos desde un archivo CSV en una tabla de la base de datos. 

1. **Lectura de datos**: Utilizamos la función `pd.read_csv()` de Pandas para leer el archivo CSV desde la URL proporcionada.
2. **Inserción en la tabla**: Usamos el método `df.to_sql()` para insertar los datos en la tabla de la base de datos SQL. Si la tabla ya existe, se agrega la nueva información sin sobrescribir los datos existentes (`if_exists='append'`).
3. **Mensajes de estado**: Después de cargar los datos, se imprime un mensaje en la consola indicando que los datos han sido cargados exitosamente.

#### 2.2.4 Carga de Datos en las Tablas

```python
for tabla, url in config['csv_urls'].items():
    cargar_csv_a_sql(url, tabla)
```


## Paso 3: Relacionar Tablas

### 3.1 Definir Relaciones

```sql
-- Relación entre customers y geo_lookup (country_code -> country)
ALTER TABLE customers
ADD CONSTRAINT FK_customers_geo_lookup
FOREIGN KEY (country_code) REFERENCES geo_lookup(country);

-- Relación entre orders y customers (customer_id -> id en customers)
ALTER TABLE orders
ADD CONSTRAINT FK_orders_customers
FOREIGN KEY (customer_id) REFERENCES customers(id);

-- Relación entre order_status y orders (order_id -> id en orders)
ALTER TABLE order_status
ADD CONSTRAINT FK_order_status_orders
FOREIGN KEY (order_id) REFERENCES orders(id);

```



### 3.2 Visualizar Relaciones

Puedes realizar consultas para ver las relaciones entre las tablas:

```sql
SELECT c.id, c.marketing_channel, o.order_id, os.status
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_status os ON o.order_id = os.order_id;
```

Esta consulta te dará una visión general de los pedidos, clientes y el estado de los pedidos.

## Paso 4: Optimización de Consultas

### 4.1 Uso de Índices

Crear índices en columnas frecuentemente consultadas, como `country_code` o `order_id`, puede mejorar el rendimiento.

```sql
CREATE INDEX idx_country_code ON customers(country_code);
```

### 4.2 Consultas Específicas

Evita utilizar `SELECT *`, ya que consume más recursos de lo necesario. Selecciona solo las columnas que necesitas:

```sql
SELECT id, marketing_channel FROM customers WHERE country_code = 'US';
```

### 4.3 Análisis de Planes de Ejecución

Usa el análisis de planes de ejecución en SQL Server para identificar consultas que pueden optimizarse.

```sql
SET STATISTICS IO ON;
SELECT * FROM orders;
```

### 4.4 Consultas LIMIT y TOP

Utiliza `TOP` o `LIMIT` para restringir el número de filas devueltas:

```sql
SELECT TOP 5 * FROM customers ORDER BY created_on DESC;
```
---

### Recursos

- [Documentación de Azure SQL Database](https://docs.microsoft.com/en-us/azure/azure-sql/)
- [Optimización de Consultas en SQL Server](https://docs.microsoft.com/en-us/sql/relational-databases/performance/performance-tuning?view=sql-server-ver15)

---

