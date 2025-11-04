# 🍜 Generador de Datos para DynamoDB - China Wok

Sistema automatizado de generación y población de datos para las tablas de DynamoDB del sistema de gestión de la cadena de restaurantes China Wok.

## 📋 Descripción

Este proyecto genera datos de prueba ficticios pero realistas para todas las tablas del sistema China Wok, manteniendo integridad referencial entre entidades. Los datos generados están validados contra esquemas JSON Schema y listos para ser importados a DynamoDB.

## ✨ Características

- ✅ **Generación automatizada** de 9 tablas interrelacionadas
- ✅ **Validación con JSON Schema** para todos los datos
- ✅ **Integridad referencial** entre tablas (pedidos → usuarios → productos)
- ✅ **Datos realistas** para Lima, Perú (direcciones, teléfonos, menú típico)
- ✅ **Población automática a DynamoDB** con manejo de errores y retry
- ✅ **Credenciales desde ~/.aws/credentials** (AWS Academy compatible)
- ✅ **Script unificado** `setup_and_run.sh` para ejecución completa

## 🚀 Inicio Rápido

### Requisitos Previos

- Python 3.7+
- AWS CLI configurado con credenciales válidas
- Cuenta de AWS con permisos para DynamoDB

### Configuración

1. **Clonar el repositorio**
```bash
git clone <repository-url>
cd Chinawok-DataGenerator
```

2. **Configurar credenciales de AWS**

Crear o editar `~/.aws/credentials`:
```ini
[default]
aws_access_key_id=YOUR_ACCESS_KEY_ID
aws_secret_access_key=YOUR_SECRET_ACCESS_KEY
aws_session_token=YOUR_SESSION_TOKEN
```

3. **Configurar variables de entorno**

Copiar `.env.example` a `.env` y editar:
```bash
cp .env.example .env
nano .env
```

Configurar nombres de tablas y credenciales del administrador:
```bash
# Nombres de tablas en DynamoDB
TABLE_LOCALES=ChinaWok-Locales
TABLE_USUARIOS=ChinaWok-Usuarios
TABLE_PRODUCTOS=ChinaWok-Productos
TABLE_EMPLEADOS=ChinaWok-Empleados
TABLE_COMBOS=ChinaWok-Combos
TABLE_PEDIDOS=ChinaWok-Pedidos
TABLE_OFERTAS=ChinaWok-Ofertas
TABLE_RESENAS=ChinaWok-Resenas
TABLE_TOKENS=ChinaWok-Tokens

# Usuario administrador único
ADMIN_EMAIL=admin@chinawok.pe
ADMIN_PASSWORD=Admin123!
ADMIN_NOMBRE=Administrador
ADMIN_APELLIDO=Sistema
```

### Ejecución

**Opción 1: Script automático (recomendado)**
```bash
bash setup_and_run.sh
```

Este script:
1. Verifica credenciales de AWS
2. Instala dependencias Python
3. Genera datos JSON
4. Puebla DynamoDB automáticamente

**Opción 2: Ejecución manual**
```bash
# Instalar dependencias
pip install -r requirements.txt

# Generar datos
python3 DataGenerator.py

# Poblar DynamoDB
python3 DataPoblator.py
```

## 📊 Estructura de Datos Generados

```
dynamodb_data/
├── locales.json           # 100 locales
├── usuarios.json          # 5,001 usuarios (1 admin + 5,000 clientes)
├── productos.json         # ~5,000 productos distribuidos por local
├── empleados.json         # 500 empleados (cocineros, repartidores, despachadores)
├── combos.json            # ~500 combos de productos
├── pedidos.json           # 10,000 pedidos en diferentes estados
├── ofertas.json           # ~500 ofertas activas
└── resenas.json           # 1,000 reseñas de pedidos completados
```

## 🗂️ Esquemas de Tablas DynamoDB

### 1. **Usuarios** (Global)
```
PK: correo
Atributos:
  - nombre: string
  - correo: email
  - contrasena: string
  - role: "Cliente" | "Admin"
  - informacion_bancaria?: {
      numero_tarjeta: string
      cvv: string
      fecha_vencimiento: string
      direccion_facturacion: string
    }
```

### 2. **Locales**
```
PK: local_id
Atributos:
  - direccion: string
  - telefono: string
  - hora_apertura: string
  - hora_finalizacion: string
```

### 3. **Productos**
```
PK: local_id
SK: nombre
Atributos:
  - precio: number
  - descripcion: string
  - categoria: enum[12 categorías]
  - stock: integer
```

### 4. **Empleados**
```
PK: local_id
SK: dni
Atributos:
  - nombre: string
  - apellido: string
  - role: "Repartidor" | "Cocinero" | "Despachador"
  - calificacion_prom: number (0-5)
  - sueldo: number
```

### 5. **Combos**
```
PK: local_id
SK: combo_id
Atributos:
  - nombre: string
  - productos_nombres: string[]
  - descripcion: string
```

### 6. **Pedidos**
```
PK: local_id
SK: pedido_id
Atributos:
  - usuario_correo: email
  - productos_nombres: string[]
  - cocinero_dni: string
  - despachador_dni: string
  - repartidor_dni: string
  - costo: number
  - status: "eligiendo" | "cocinando" | "empacando" | "enviando" | "recibido"
  - fecha_entrega?: datetime (solo si status = enviando/recibido)
  - direccion?: string (solo si status = enviando/recibido)
```

### 7. **Ofertas**
```
PK: local_id
SK: oferta_id
Atributos:
  - producto_nombre?: string
  - combo_id?: string
  - fecha_inicio: datetime
  - fecha_limite: datetime
  - porcentaje_descuento: number
```

### 8. **Reseñas**
```
PK: local_id
SK: resena_id
Atributos:
  - pedido_id: string
  - resena?: string
  - calificacion: number (0-5)
  - empleados_dni: string[] (0-3 empleados)
```

### 9. **Tokens**
```
PK: token
Atributos:
  - correo_usuario: string
  - fecha_creacion: datetime
  - expiracion: datetime
```
## 🏗️ Arquitectura del Proyecto

```
Chinawok-DataGenerator/
├── data_generator_utils/
│   ├── __init__.py
│   ├── config.py              # Configuración centralizada
│   ├── utils.py               # Utilidades (emails, passwords, tarjetas)
│   ├── sample_data.py         # Datos de muestra (nombres, direcciones)
│   └── generators/
│       ├── __init__.py
│       ├── locales_generator.py
│       ├── usuarios_generator.py
│       ├── productos_generator.py
│       ├── empleados_generator.py
│       ├── combos_generator.py
│       ├── pedidos_generator.py
│       ├── ofertas_generator.py
│       └── resenas_generator.py
├── schemas-validation/         # JSON Schemas para validación
│   ├── usuarios.json
│   ├── locales.json
│   ├── productos.json
│   ├── empleados.json
│   ├── combos.json
│   ├── pedidos.json
│   ├── ofertas.json
│   └── resenas.json
├── dynamodb_data/             # Datos JSON generados (creado al ejecutar)
├── DataGenerator.py           # Script principal de generación
├── DataPoblator.py            # Script de población a DynamoDB
├── setup_and_run.sh           # Script automatizado completo
├── requirements.txt           # Dependencias Python
├── .env.example               # Ejemplo de configuración
└── README.md
```

## 🔧 Configuración Avanzada

### Modificar Cantidades de Datos

Editar `data_generator_utils/config.py`:

```python
class Config:
    # Cantidades de registros
    NUM_LOCALES = 100
    NUM_USUARIOS = 5000
    NUM_EMPLEADOS = 500
    NUM_PEDIDOS = 10000
    NUM_OFERTAS_POR_LOCAL = 5
    NUM_RESENAS = 1000
    
    # Porcentajes
    PORCENTAJE_USUARIOS_CON_TARJETA = 0.7
    
    # Rangos de precios
    PRECIO_MIN_PRODUCTO = 10.0
    PRECIO_MAX_PRODUCTO = 50.0
```

### Agregar Nuevas Categorías de Productos

Editar `data_generator_utils/config.py`:

```python
CATEGORIAS_PRODUCTO = [
    "Arroces",
    "Tallarines",
    "Pollo al wok",
    "Carne de res",
    "Cerdo",
    "Mariscos",
    "Entradas",
    "Guarniciones",
    "Sopas",
    "Combos",
    "Bebidas",
    "Postres",
    # Agregar nuevas categorías aquí
]
```
