## 🚀 Documentar una API GraphQL Ruby con SpectaQL

### 1️⃣ Preparar el entorno

1. Instala dependencias del backend:

```bash
bundle install
```

1. Levanta el servidor Rails (por defecto quedará en `http://localhost:3000`):

```bash
rails s
```

1. Confirma el endpoint GraphQL en `config/routes.rb`
    
    Normalmente verás algo como:
    

```ruby
post "/graphql", to: "graphql#execute"
mount GraphiQL::Rails::Engine, at: "/graphiql", graphql_path: "/graphql"
```

→ El endpoint real para introspección es: **`http://localhost:3000/graphql`**.

---

### 2️⃣ Obtener el schema GraphQL

Tienes dos maneras:

### 🔹 A) Descargar archivo local (recomendado para versionar, pero puede generar problemas)

```bash
npm install -g get-graphql-schema
get-graphql-schema http://localhost:3000/graphql > schema.graphql
```

### 🔹 B) Usar la URL directa (sin archivo, no es necesario el archivo intermedio)

SpectaQL puede conectarse directamente a la API si el servidor está levantado.

---

### 3️⃣ Crear archivo de configuración SpectaQL

Guarda esto como `spectaql.yml` en la raíz del proyecto:

```yaml
introspection:
  # Opción A: si descargaste el schema
  #file: ./schema.graphql
  # Opción B: si prefieres conectarte directo
  url: http://localhost:3000/graphql

info:
  title: "API GraphQL C1DO1"
  description: |
    Documentación generada automáticamente a partir del schema GraphQL.
    Incluye todos los tipos, queries, mutations y descripciones declaradas en el backend.
  contact:
    name: "Equipo QA / Dev"
    email: "qa@c1do1.ai"
  license:
    name: "Propietario interno"
    url: "https://c1do1.ai"

servers:
  - url: "http://localhost:3000/graphql"
    description: "Servidor local de desarrollo"

settings:
  grouping:
    query: "Queries"
    mutation: "Mutations"
    subscription: "Subscriptions"
  examples: true
  deprecations: true
```

---

### 4️⃣ Instalar y ejecutar SpectaQL

1. Instalar SpectaQL (global o como dev dependency):

```bash
npm install -g spectaql
```

1. Generar la documentación:

```bash
spectaql spectaql.yml
```

Esto crea una carpeta **`public/`** con un sitio web estático.

---

### 5️⃣ Visualizar y compartir

- Abrir localmente:

```bash
open public/index.html    # macOS
xdg-open public/index.html # Linux o wsl
start public\index.html    # Windows
```

- Para compartir:
    - Subir `public/` a un hosting estático (GitHub Pages, Netlify, Vercel, intranet).
    - O mantenerlo en el repo como `/docs` y abrirlo manualmente.

---

### 6️⃣ Mantener actualizada la documentación

Cada vez que cambie el schema:

```bash
get-graphql-schema http://localhost:3000/graphql > schema.graphql
spectaql spectaql.yml
```

Si agregas descripciones a tus tipos y campos en Ruby:

```ruby
class Types::UserType < Types::BaseObject
  description "Usuario de la plataforma"
  field :id, ID, null: false, description: "Identificador único"
  field :email, String, null: false, description: "Correo electrónico del usuario"
end
```

→ Al regenerar, aparecerán automáticamente en la documentación.

---

### 🔥 Resumen rápido de comandos clave

```bash
# Instalar utilidades
npm install -g get-graphql-schema spectaql

# Arrancar backend Rails
rails s

# Descargar schema
get-graphql-schema http://localhost:3000/graphql > schema.graphql

# Generar docs
spectaql spectaql.yml
```

## 🌐 Documentación de la API GraphQL

La documentacion fue añadida a siguiente repositorio:
👉 [Repo C1DO1 API Docs](https://github.com/ChupeteBecker/C1DO1-db-docs.git)

La documentación generada con [SpectaQL](https://github.com/anvilco/spectaql) está disponible públicamente en:

👉 [C1DO1 API Docs](https://c1do1-api-docs.netlify.app/)

Este sitio contiene:

- **Queries, Mutations y Subscriptions** disponibles en la API.
- Descripciones de cada tipo, campo y argumento definidos en el esquema.
- Ejemplos de uso y detalles sobre campos deprecados.

Cada vez que se actualice el schema GraphQL, recuerda:

```bash
# Regenerar la documentación
spectaql spectaql.yml

# Subir cambios
git add .
git commit -m "docs: actualizar documentación GraphQL"
git push
```

## 🧩 ¿Qué es GraphQL y cómo funciona?

**GraphQL** es un lenguaje de consultas para APIs y un *runtime* que permite a los clientes pedir exactamente los datos que necesitan, ni más ni menos.

Fue desarrollado por Facebook y hoy es un estándar abierto usado ampliamente en aplicaciones web y móviles.

### 🔎 Conceptos clave

- **Schema (esquema):**
    
    Define los tipos de datos, sus campos y las operaciones disponibles.
    
    Actúa como contrato entre el frontend y el backend.
    
- **Types (Tipos):**
    
    Representan los objetos y estructuras de datos (por ejemplo `User`, `Course`, `MutationResponse`).
    
- **Queries:**
    
    Operaciones para **leer** información. El cliente especifica qué campos quiere recibir.
    
- **Mutations:**
    
    Operaciones para **crear, actualizar o eliminar** datos en el servidor.
    
- **Subscriptions:**
    
    Permiten recibir datos en tiempo real cuando ocurren cambios en el servidor.
    
- **Introspection:**
    
    Funcionalidad integrada que permite descubrir automáticamente la estructura de la API (ideal para herramientas de documentación y autocompletado).
    

### ⚡ Flujo de funcionamiento

1. **Definición del Schema**
    
    El backend (en este caso Ruby con la gema [`graphql-ruby`](https://graphql-ruby.org/)) define tipos, queries y mutations usando clases y campos con descripciones.
    
2. **Petición desde el Cliente**
    
    El frontend envía una consulta en formato GraphQL, indicando exactamente los campos que necesita.
    
    Ejemplo:
    
    ```graphql
    query {
      user(id: 1) {
        id
        email
        profile {
          name
          avatarUrl
        }
      }
    }
    
    ```
    

1. **Resolución en el Servidor**
    
    El servidor recibe la consulta y, basándose en el schema, ejecuta resolvers (métodos Ruby) que obtienen los datos desde la base de datos u otros servicios.
    
2. **Respuesta Precisa**
    
    El servidor responde solo con los campos solicitados:
    
    ```json
    {
      "data": {
        "user": {
          "id": "1",
          "email": "user@example.com",
          "profile": {
            "name": "Nombre Apellido",
            "avatarUrl": "https://cdn.example.com/avatar.png"
          }
        }
      }
    }
    
    ```