# 📘 Documentación Técnica  
## Estándares de Arquitectura, Nomenclatura y Desarrollo  
### Backend – FastAPI

---

## 1. Objetivo

Definir estándares técnicos obligatorios para el desarrollo backend en FastAPI, garantizando:

- Mantenibilidad
- Escalabilidad
- Consistencia
- Compatibilidad con Laravel
- Claridad técnica

---

## 2. Principios de Arquitectura

1. El estándar pertenece al sistema, no al framework
2. La base de datos es la fuente de verdad
3. FastAPI no impone convenciones → el equipo sí
4. Separación estricta de responsabilidades
5. Bajo acoplamiento, alta cohesión
6. Código predecible > código creativo

---

## 3. Estructura del Proyecto

```bash
app/
├── core/
│   ├── config.py
│   ├── constants.py
│   ├── database.py
│   ├── security.py
│   └── exceptions.py
│
├── models/
│   ├── user.py
│   ├── company.py
│   └── user_company.py
│
├── schemas/
│   ├── user_schema.py
│   └── company_schema.py
│
├── repositories/
│   └── user_repository.py
│
├── services/
│   └── user_service.py
│
├── api/
│   └── v1/
│       └── user_controller.py
│
├── utils/
│   └── date_utils.py
│
└── main.py
```

## 4. Convenciones de Base de Datos
### 4.1 Tablas
- Plural
- snake_case
- Sin prefijos

```bash
users
companies
orders
user_company
```

### 4.2 Columnas
- snake_case
- Semánticas
- Sin abreviaciones

```bash
user_name
email
created_at
updated_at
```

### 4.3 Primary Keys
Nombre fijo: id

```bash
id INTEGER PRIMARY KEY
```

### 4.4 Foreign Keys
Formato obligatorio: <tabla_singular>_id


Ejemplos:

sql

```bash
user_id
company_id
order_id
```

❌ Prohibido:
- id_user
- user_pk
- fk_user

### 4.5 Tablas Pivote (Many-to-Many)
Formato: singular_singular

Ejemplo:

```bash
user_company
```


Campos mínimos:
```bash
user_id
company_id
created_at
```


Campos adicionales permitidos:

```bash
role
is_active
```


❌ Nunca usar pivot en el nombre de la tabla

## 5. Convenciones de Código
### 5.1 Variables
camelCase

```bash
const userName = "Jhon";
```

### 5.2 Constantes
- UPPER_SNAKE_CASE
- Centralizadas

```bash
MAX_LOGIN_ATTEMPTS = 5
DEFAULT_TIMEZONE = "UTC"
JWT_EXPIRATION_MINUTES = 60
```


Ubicación:

```bash
app/core/constants.py
```

5.3 Clases
- PascalCase
- Singular

```bash
class UserService:
    pass
```

    
5.4 Funciones y Métodos
- snake_case
- Verbos descriptivos

```bash
def get_user_by_id(user_id: int):
    pass
```

### 6. Modelos (SQLAlchemy)
- Ejemplo: User

```bash
class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    user_name = Column(String, nullable=False)
    email = Column(String, unique=True)
    created_at = Column(DateTime)
    updated_at = Column(DateTime)
```

6.1 Relaciones Many-to-Many
- python
- 
```bash
class UserCompany(Base):
    __tablename__ = "user_company"

    user_id = Column(Integer, ForeignKey("users.id"), primary_key=True)
    company_id = Column(Integer, ForeignKey("companies.id"), primary_key=True)
    role = Column(String)
    created_at = Column(DateTime)
```

## 7. Schemas (Pydantic)
### 7.1 Regla General
- Base de datos → snake_case
- API / Frontend → camelCase
- Uso obligatorio de alias

```bash
class UserSchema(BaseModel):
    userName: str = Field(alias="user_name")
    email: str

    class Config:
        populate_by_name = True
        from_attributes = True
```

## 8. Repositories
### 8.1 Responsabilidad
- Acceso a base de datos
- Sin lógica de negocio
- Sin validaciones

```bash
class UserRepository:

    def get_by_id(self, db, user_id: int):
        return db.query(User).filter(User.id == user_id).first()
```

## 9. Services
### 9.1 Responsabilidad
- Reglas de negocio
- Validaciones
- Orquestación

```bash
class UserService:

    def get_user(self, db, user_id: int):
        user = UserRepository().get_by_id(db, user_id)
        if not user:
            raise DomainError("User not found")
        return user
```

## 10. Controladores (API)
### 10.1 Responsabilidad
- Manejo HTTP
- Seguridad
- Request / Response

```bash
@router.get("/users/{user_id}", response_model=UserSchema)
def get_user(user_id: int, db=Depends(get_db)):
    return UserService().get_user(db, user_id)
```

### 11. Utilidades
- Funciones puras
- Reutilizables
- Sin lógica de negocio

```bash
def format_date_to_utc(date):
    return date.astimezone(timezone.utc)
```

## 12. Manejo de Errores
### 12.1 Errores de Dominio
```bash
class DomainError(Exception):
    pass
```

### 12.2 Errores HTTP
```bash
raise HTTPException(status_code=404, detail="User not found")
```

## 13. Seguridad
- JWT obligatorio
- Tokens con expiración
- Password hashing con bcrypt
- Secrets solo en .env

❌ Prohibido hardcodear credenciales

## 14. Versionado de API
- Versionado por URL
- No romper versiones existentes

```bash
/api/v1/users
/api/v2/users
```

## 15. Regla Final
Todo código que no cumpla este documento no se integra al repositorio.








# 📘 Estándar de API REST - Documentación Técnica (ES)

## 🧱 1. Estructura general de respuesta

### ✅ Éxito
```json
{
  "success": true,
  "status": {
    "code": 200,
    "message": "OK",
    "description": "Operación exitosa."
  },
  "result": {
    "items": [
      { "id": 1, "name": "Running Shoes", "price": 120.5 },
      { "id": 2, "name": "Tennis Classic", "price": 89.9 }
    ],
    "pagination": {
      "total": 42,
      "count": 2,
      "per_page": 5,
      "current_page": 2,
      "total_pages": 9,
      "links": {
        "previous": "/api/v1/catalogue/shoes?page=1",
        "next": "/api/v1/catalogue/shoes?page=3"
      }
    }
  }
}
```

### ❌ Error
```json
{
  "success": false,
  "status": {
    "code": 404,
    "message": "Resource not found",
    "description": "El recurso no se encuentra en el servidor."
  },
  "error": {
    "type": "NotFoundError",
    "details": "User not found with ID 999"
  }
}
```

---

## 🔐 2. Autenticación (Bearer Token - JWT)

**Header obligatorio:**
```
Authorization: Bearer <access_token>
```

**Ejemplo:**
```bash
curl -X GET https://api.example.com/api/v1/user/profile   -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."   -H "Accept: application/json"
```

**Ejemplo de respuesta:**
```json
{
  "success": true,
  "status": { "code": 200, "message": "Authenticated", "description": "Autenticación exitosa." },
  "result": {
    "user": { "id": 1, "name": "John Doe", "email": "john@example.com" },
    "token": {
      "access_token": "eyJhbGciOi...",
      "token_type": "Bearer",
      "expires_in": 3600
    }
  }
}
```

---

## 📦 3. Paginación y filtros

**Parámetros comunes:**
- `page`: número de página (default: 1)
- `limit`: cantidad de resultados (default: 10)
- `sort`: campo de ordenamiento (`-created_at` para descendente)
- `filter`: criterios de filtrado

**Ejemplo:**
```
GET /api/v1/catalogue/shoes?page=2&limit=5&sort=-price
```

**Respuesta:**
```json
{
  "success": true,
  "status": { "code": 200, "message": "OK", "description": "Operación exitosa." },
  "result": {
    "items": [
      { "id": 1, "name": "Running Shoes", "price": 120.5 },
      { "id": 2, "name": "Tennis Classic", "price": 89.9 }
    ],
    "pagination": {
      "total": 42,
      "count": 2,
      "per_page": 5,
      "current_page": 2,
      "total_pages": 9,
      "links": {
        "previous": "/api/v1/catalogue/shoes?page=1",
        "next": "/api/v1/catalogue/shoes?page=3"
      }
    }
  }
}
```

---

## 🧭 4. Versionamiento

Opciones válidas:

- En URL: `/api/v1/...`
- En Header:
  ```
  X-API-Version: 1.2.3
  ```

---

## ⚙️ 5. Headers comunes

| Header | Obligatorio | Descripción |
|--------|--------------|-------------|
| `Authorization` | Sí | Bearer token JWT |
| `Accept` | Sí | Formato esperado, ej. `application/json` |
| `X-API-Version` | Opcional | Versión actual de cliente o app |
| `X-Request-ID` | Opcional | ID único de petición para trazabilidad |
| `Accept-Language` | Opcional | Idioma preferido (`es`, `en`, `fr`) |
| `Content-Type` | Sí | Tipo de contenido, ej. `application/json` |
| `Lang` | Opcional por defecto tomara es | Idioma preferido (`es`, `en`, `fr`) |   
| `Timezone` | Opcional | Zona horaria preferida(`America/Caracas`) |   

---

# 🌐 Estándar de API REST

## 🧩 6. Ejemplo de endpoint genérico y estructura REST

Los endpoints de una API definen las **rutas de acceso a los recursos** del sistema.  
Cada ruta representa una acción o conjunto de acciones sobre un recurso específico (usuarios, productos, pedidos, etc.).  

### Ejemplo rápido

| Método | Ruta | Descripción |
|--------|------|-------------|
| **GET** | `/api/admin/user/profile` | Retorna los datos del usuario autenticado. |
| **GET** | `/api/admin/catalogue/shoes` | Lista los productos disponibles (con paginación). |
| **POST** | `/api/admin/orders` | Crea una nueva orden de compra. |
| **DELETE** | `/api/admin/orders/{id}` | Elimina una orden existente. |

---

## 🔧 Estructura general de rutas en una API REST

Una API bien estructurada sigue el siguiente patrón:

```
/api/{version}/{recurso}/{acción}
```

### Desglose
- **/api** → prefijo común que indica que es una API.  
- **{version}** → versión del servicio (por ejemplo, `v1`) que permite mantener compatibilidad cuando se actualiza la API.  
- **{recurso}** → el nombre del objeto o entidad principal (por ejemplo, `users`, `orders`, `shoes`).  
- **{acción}** → opcional. Se usa solo cuando la operación no encaja con los verbos HTTP (por ejemplo, `/api/v1/users/{id}/activate`).  

### Verbos HTTP y su propósito

| Método | Descripción | Ejemplo |
|--------|--------------|---------|
| **GET** | Obtiene recursos (uno o varios). | `/api/v1/shoes` |
| **POST** | Crea un nuevo recurso. | `/api/v1/orders` |
| **PUT** | Reemplaza completamente un recurso existente. | `/api/v1/shoes/{id}` |
| **PATCH** | Actualiza parcialmente un recurso. | `/api/v1/shoes/{id}` |
| **DELETE** | Elimina un recurso. | `/api/v1/orders/{id}` |

---

## 👟 Ejemplo con el recurso `shoes`

### 1️⃣ Rutas públicas (accesibles sin autenticación)
Estas rutas permiten consultar información general sin autenticarse.  

| Método | Ruta | Descripción |
|--------|------|-------------|
| **GET** | `/api/v1/shoes` | Lista todos los zapatos. |
| **GET** | `/api/v1/shoes/:id` | Obtiene los detalles de un zapato específico. |
| **GET** | `/api/v1/shoes/search?q=nike` | Busca zapatos por nombre o categoría. |

### 2️⃣ Rutas privadas (requieren autenticación)
Diseñadas para administradores o usuarios con permisos específicos.  
Se recomienda agruparlas bajo `/api/v1/admin/` para dejar claro su propósito.

| Método | Ruta | Descripción |
|--------|------|-------------|
| **GET** | `/api/v1/admin/shoes` | Lista todos los zapatos con información completa. |
| **POST** | `/api/v1/admin/shoes` | Crea un nuevo zapato. |
| **PUT** | `/api/v1/admin/shoes/:id` | Actualiza completamente un zapato. |
| **DELETE** | `/api/v1/admin/shoes/:id` | Elimina un zapato existente. |

---

## 🧱 Estructura sólida recomendada

Organización de rutas por roles y versiones:

```
/api/v1/
├── shoes/               → Público
│   ├── GET /            → Lista general
│   ├── GET /:id         → Detalle
│   └── GET /search      → Búsqueda
└── admin/               → Privado
    └── shoes/
        ├── POST /       → Crear
        ├── PUT /:id     → Editar
        ├── DELETE /:id  → Eliminar
        └── GET /        → Lista completa
```

---

## ⚙️ Uso correcto de códigos HTTP

Cada endpoint debe devolver **códigos HTTP reales** para indicar el estado de la operación.  
Esto facilita el manejo de errores en el cliente y mejora la depuración.

| Código | Nombre | Cuándo usarlo | Ejemplo |
|--------|---------|----------------|----------|
| **200 OK** | Éxito genérico | Cuando la solicitud fue exitosa y se devuelve información. | `GET /api/v1/shoes` |
| **201 Created** | Creación exitosa | Cuando se crea un nuevo recurso. | `POST /api/v1/orders` |
| **204 No Content** | Sin contenido | Cuando se elimina un recurso correctamente sin respuesta adicional. | `DELETE /api/v1/orders/{id}` |
| **400 Bad Request** | Error del cliente | Cuando los parámetros o el cuerpo de la solicitud son inválidos. | Falta un campo obligatorio |
| **401 Unauthorized** | No autenticado | Cuando el token o credenciales son inválidas o ausentes. | Token expirado |
| **403 Forbidden** | Sin permisos | El usuario está autenticado, pero no tiene permisos para acceder. | Usuario común intenta acceder a `/admin` |
| **404 Not Found** | Recurso no encontrado | Cuando el recurso solicitado no existe. | `GET /api/v1/shoes/999` |
| **409 Conflict** | Conflicto de estado | Cuando hay un conflicto lógico (por ejemplo, crear un duplicado). | `POST` con correo existente |
| **422 Unprocessable Entity** | Error de validación | Cuando los datos son válidos en formato, pero no cumplen las reglas del negocio. | Campos inválidos |
| **500 Internal Server Error** | Error del servidor | Cuando ocurre una excepción no controlada. | Falla en base de datos |

## 🧾 Ejemplos de respuesta con código y formato estándar

---

### ✅ **200 – OK (Éxito genérico)**
```json
{
  "success": true,
  "status": {
    "code": 200,
    "message": "OK",
    "description": "Zapatos listados correctamente."
  },
  "result": [
    "items": [
      { "id": 1, "name": "Running Shoes", "price": 120.5 },
      { "id": 2, "name": "Tennis Classic", "price": 89.9 }
    ],
    "pagination": {
      "total": 42,
      "count": 2,
      "per_page": 5,
      "current_page": 2,
      "total_pages": 9,
      "links": {
        "previous": "/api/v1/catalogue/shoes?page=1",
        "next": "/api/v1/catalogue/shoes?page=3"
      }
    }
  ]
}

```

---

### 🎉 **201 – Created (Recurso creado exitosamente)**
```json
{
  "success": true,
  "status": {
    "code": 201,
    "message": "Created",
    "description": "Recurso creado exitosamente."
  },
  "result": {
    "id": 35,
    "name": "Puma Classic",
    "price": 89.99
  }
}

```

---

### 🧹 **204 – No Content (Eliminación o actualización sin respuesta)**
> No se devuelve cuerpo de respuesta.  
> Solo se responde con el código de estado HTTP `204`:
```
HTTP/1.1 204 No Content
```

---

### ⚠️ **400 – Bad Request (Error de formato o parámetros)**
```json
{
  "success": false,
  "status": {
    "code": 400,
    "message": "Bad Request",
    "description": "Error de formato o parámetros"
  },
  "errors": {
    "name": ["El campo 'name' es obligatorio."]
  }
}

```

---

### 🔒 **401 – Unauthorized (No autenticado)**
```json
{
  "success": false,
  "status": {
    "code": 401,
    "message": "Unauthorized",
    "description": "No tienes permisos para acceder a este recurso."
  }
}

```

---

### 🚫 **403 – Forbidden (Sin permisos)**
```json
{
  "success": false,
  "status": {
    "code": 403,
    "message": "Forbidden",
    "description": "No tienes permisos para acceder a este recurso."
  }
}

```

---

### 🔍 **404 – Not Found (Recurso no encontrado)**
```json
{
  "success": false,
  "status": {
    "code": 404,
    "message": "Not Found",
    "description": "El recurso no se encuentra en el servidor."
  }
}
```

---

### ⚔️ **409 – Conflict (Conflicto de estado o duplicado)**
```json
{
  "success": false,
  "status": {
    "code": 409,
    "message": "Conflict",
    "description": "El recurso ya existe o hay un conflicto con los datos enviados."
  }
}

```

---

### 🚧 **422 – Unprocessable Entity (Error de validación)**
```json
{
  "success": false,
  "status": {
    "code": 422,
    "message": "Unprocessable Entity",
    "description": "Error de validación"
  },
  "errors": {
    "price": ["El campo 'price' debe ser un número positivo."]
  }
}

```

---

### 💥 **500 – Internal Server Error (Error del servidor)**
```json
{
  "success": false,
  "status": {
    "code": 500,
    "message": "Internal Server Error",
    "description": "Ocurrió un error inesperado en el servidor. Intenta nuevamente más tarde. Si continua contactanos a este correo: email@example.com"
  }
}

```


