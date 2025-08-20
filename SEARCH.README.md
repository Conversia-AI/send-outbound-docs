# API de Búsqueda de Contactos Externa - Guía Completa para Desarrolladores

## 📜 Descripción General

La API de Búsqueda de Contactos Externa permite a sistemas CRM externos buscar contactos con capacidades avanzadas de filtrado. Este endpoint está diseñado específicamente para integraciones externas y requiere autenticación con clave API y restricciones por unidad organizacional.

- **URL Base:** `https://engine.conversia.ai`
- **Endpoint:** `GET /api/v1/contacts/search-external`

---

## 🤔 ¿Qué hace este endpoint?

Este endpoint te permite:

- 🔍 Buscar contactos por nombre, email, teléfono, empresa y `ref_id`.
- 📊 Filtrar por campos personalizados con operadores avanzados.
- 📄 Paginar resultados para manejar grandes volúmenes de datos.
- 🗂️ Ordenar resultados por diferentes campos.
- 📅 Filtrar por rangos de fechas de creación/actualización.
- 🔒 Acceso seguro con autenticación API.

---

## 🔑 Autenticación (OBLIGATORIA)

### Headers Requeridos

> **IMPORTANTE:** Sin estos headers, la API rechazará tu solicitud con un error `403 Forbidden`.

| Header             | Obligatorio | Descripción                          | Ejemplo                                |
| :----------------- | :---------- | :----------------------------------- | :------------------------------------- |
| `X-API-KEY`        | ✅ Sí       | Clave API externa para autenticación | `01989b2c-48d2-7a9e-8b89-14e4a6377dee` |
| `X-API-ORGUNIT-ID` | ✅ Sí       | ID de unidad organizacional (UUID)   | `31f1d8e5-9c9e-409d-8336-cf7da99921bb` |

#### Ejemplo de Headers

```http
X-API-KEY: 01989b2c-48d2-7a9e-8b89-14e4a6377dee
X-API-ORGUNIT-ID: 31f1d8e5-9c9e-409d-8336-cf7da99921bb
```

### ¿Dónde obtengo estos valores?

- **`X-API-KEY`**: Solicítala a tu administrador de Conversia o genérala desde el panel de administración.
- **`X-API-ORGUNIT-ID`**: Es el UUID de tu organización, disponible en la sección de configuración de tu cuenta.

---

## ⚙️ Parámetros de Consulta (Query Parameters)

### 1. Búsqueda Básica

| Parámetro        | Tipo   | Obligatorio | Descripción                                                    | Ejemplo                    |
| :--------------- | :----- | :---------- | :------------------------------------------------------------- | :------------------------- |
| `search`         | string | No          | Búsqueda general en nombre, email, teléfono, empresa, `ref_id` | `search=juan`              |
| `id`             | string | No          | ID exacto del contacto (UUID)                                  | `id=e84b12c1-1dcb-4a77...` |
| `name`           | string | No          | Búsqueda parcial en nombre (insensible a mayúsculas)           | `name=García`              |
| `email`          | string | No          | Email exacto (normalizado)                                     | `email=juan@empresa.com`   |
| `phone`          | string | No          | Búsqueda parcial en teléfono                                   | `phone=555`                |
| `ref_id`         | string | No          | ID de referencia externa/CRM                                   | `ref_id=CRM-123`           |
| `company`        | string | No          | Búsqueda parcial en empresa                                    | `company=Acme`             |
| `session_status` | string | No          | Estado de sesión/engagement                                    | `session_status=active`    |

### 2. Paginación

| Parámetro   | Tipo    | Obligatorio | Por Defecto | Límite | Descripción                    |
| :---------- | :------ | :---------- | :---------- | :----- | :----------------------------- |
| `page`      | integer | No          | 1           | -      | Número de página (inicia en 1) |
| `page_size` | integer | No          | 20          | 100    | Resultados por página          |

**Ejemplo de paginación:**

```http
GET /api/v1/contacts/search-external?page=2&page_size=50
```

### 3. Ordenamiento

| Parámetro    | Tipo   | Valores Permitidos                                                              | Descripción                    |
| :----------- | :----- | :------------------------------------------------------------------------------ | :----------------------------- |
| `sort_by`    | string | `id`, `name`, `email`, `phone`, `company`, `ref_id`, `created_at`, `updated_at` | Campo por el cual ordenar      |
| `sort_order` | string | `asc`, `desc`                                                                   | Orden ascendente o descendente |

**Ejemplo de ordenamiento:**

```http
GET /api/v1/contacts/search-external?sort_by=created_at&sort_order=desc
```

### 4. Filtros de Fecha

> Utiliza el formato **ISO 8601 / RFC3339** para las fechas (`YYYY-MM-DDTHH:MM:SSZ`).

| Parámetro        | Formato  | Descripción                                  | Ejemplo                               |
| :--------------- | :------- | :------------------------------------------- | :------------------------------------ |
| `created_after`  | ISO 8601 | Contactos creados después de esta fecha      | `created_after=2024-01-01T00:00:00Z`  |
| `created_before` | ISO 8601 | Contactos creados antes de esta fecha        | `created_before=2024-12-31T23:59:59Z` |
| `updated_after`  | ISO 8601 | Contactos actualizados después de esta fecha | `updated_after=2024-06-01T00:00:00Z`  |
| `updated_before` | ISO 8601 | Contactos actualizados antes de esta fecha   | `updated_before=2024-06-30T23:59:59Z` |

**Ejemplo de filtro por fechas:**

```http
GET /api/v1/contacts/search-external?created_after=2024-05-01T00:00:00Z&created_before=2024-06-01T00:00:00Z
```

### 5. Campos Personalizados (Custom Fields) - AVANZADO

Los campos personalizados son dinámicos y específicos de tu organización. Permiten filtros muy potentes.

#### Sintaxis

```
cf.<nombre_campo>[__<operador>]=<valor>
```

o

```
custom.<nombre_campo>[__<operador>]=<valor>
```

#### Operadores Disponibles

| Operador           | Significado           | Ejemplo                         | Descripción                                  |
| :----------------- | :-------------------- | :------------------------------ | :------------------------------------------- |
| `eq` (por defecto) | Igual a               | `cf.estado=activo`              | Valor exacto                                 |
| `neq`              | No igual a            | `cf.estado__neq=inactivo`       | Diferente al valor                           |
| `contains`         | Contiene              | `cf.notas__contains=demo`       | Contiene substring                           |
| `icontains`        | Contiene (insensible) | `cf.ciudad__icontains=lima`     | Contiene substring sin importar mayúsculas   |
| `gt`               | Mayor que             | `cf.edad__gt=25`                | Para números                                 |
| `gte`              | Mayor o igual         | `cf.puntuacion__gte=90`         | Para números                                 |
| `lt`               | Menor que             | `cf.puntuacion__lt=100`         | Para números                                 |
| `lte`              | Menor o igual         | `cf.saldo__lte=0`               | Para números                                 |
| `in`               | En lista              | `cf.rol__in=comprador,vendedor` | Valores separados por coma                   |
| `nin`              | No en lista           | `cf.tipo__nin=lead,archivado`   | Excluir valores                              |
| `exists`           | Campo existe          | `cf.meta__exists`               | Solo verifica existencia (no requiere valor) |

#### Ejemplos de Campos Personalizados

- **Filtro simple:** `cf.estado=activo`
- **Filtros múltiples:** `cf.estado=activo&cf.region=LATAM&cf.puntuacion__gte=85`
- **Filtro por lista:** `cf.etiquetas__in=vip,premium,oro`
- **Filtro de rango numérico:** `cf.edad__gte=25&cf.edad__lte=65`
- **Verificar existencia de campo:** `cf.notas_especiales__exists`

---

## 🚀 Ejemplos Prácticos de Uso

### 1. Búsqueda Básica

**Caso:** Buscar todos los contactos que contengan "María".

```http
GET /api/v1/contacts/search-external?search=María
Host: engine.conversia.ai
X-API-KEY: 01989b2c-48d2-7a9e-8b89-14e4a6377dee
X-API-ORGUNIT-ID: 31f1d8e5-9c9e-409d-8336-cf7da99921bb
```

### 2. Búsqueda con Paginación y Ordenamiento

**Caso:** Obtener la segunda página de resultados, 10 por página, ordenados por fecha de creación descendente.

```http
GET /api/v1/contacts/search-external?page=2&page_size=10&sort_by=created_at&sort_order=desc
Host: engine.conversia.ai
X-API-KEY: 01989b2c-48d2-7a9e-8b89-14e4a6377dee
X-API-ORGUNIT-ID: 31f1d8e5-9c9e-409d-8336-cf7da99921bb
```

### 3. Filtros por Campos Personalizados

**Caso:** Encontrar contactos activos de la región "US" con una puntuación mayor o igual a 85.

```http
GET /api/v1/contacts/search-external?cf.estado=activo&cf.region=US&cf.puntuacion__gte=85
Host: engine.conversia.ai
X-API-KEY: 01989b2c-48d2-7a9e-8b89-14e4a6377dee
X-API-ORGUNIT-ID: 31f1d8e5-9c9e-409d-8336-cf7da99921bb
```

### 4. Búsqueda Avanzada Combinada

**Caso:** Buscar contactos con etiquetas "vip" o "premium", creados en mayo de 2024, y ordenarlos por nombre.

```http
GET /api/v1/contacts/search-external?cf.etiquetas__in=vip,premium&created_after=2024-05-01T00:00:00Z&created_before=2024-05-31T23:59:59Z&sort_by=name&sort_order=asc
Host: engine.conversia.ai
X-API-KEY: 01989b2c-48d2-7a9e-8b89-14e4a6377dee
X-API-ORGUNIT-ID: 31f1d8e5-9c9e-409d-8336-cf7da99921bb
```

### 5. Filtro por Empresa y Estado

**Caso:** Encontrar contactos de empresas que contengan "Tech" y que estén activos, ordenados por la fecha de última actualización.

```http
GET /api/v1/contacts/search-external?company=Tech&cf.estado=activo&sort_by=updated_at&sort_order=desc
Host: engine.conversia.ai
X-API-KEY: 01989b2c-48d2-7a9e-8b89-14e4a6377dee
X-API-ORGUNIT-ID: 31f1d8e5-9c9e-409d-8336-cf7da99921bb
```

---

## 📦 Formato de Respuesta

### Respuesta Exitosa (`200 OK`)

```json
{
  "data": [
    {
      "id": "e84b12c1-1dcb-4a77-8d9a-6137f2b8c9d0",
      "name": "María García López",
      "email": "maria.garcia@empresa.com",
      "phone": "+34-666-123-456",
      "company": "Empresa Tecnológica S.L.",
      "ref_id": "CRM-2024-001",
      "custom_fields": {
        "estado": "activo",
        "region": "EMEA",
        "puntuacion": 92,
        "etiquetas": ["vip", "premium"],
        "ultima_interaccion": "2024-06-15",
        "fuente": "web"
      },
      "created_at": "2024-05-30T14:23:01Z",
      "updated_at": "2024-06-01T15:18:29Z"
    },
    {
      "id": "f95c23d2-2edc-5b88-9e0b-7248g3c9e1f1",
      "name": "Juan Pérez Martín",
      "email": "juan.perez@otrodominio.com",
      "phone": "+34-677-987-654",
      "company": "Consultoría ABC",
      "ref_id": "CRM-2024-002",
      "custom_fields": {
        "estado": "prospecto",
        "region": "LATAM",
        "puntuacion": 78,
        "etiquetas": ["nuevo", "interesado"],
        "fuente": "referido"
      },
      "created_at": "2024-06-02T09:15:30Z",
      "updated_at": "2024-06-02T09:15:30Z"
    }
  ],
  "meta": {
    "page": 1,
    "page_size": 20,
    "total": 147,
    "execution_time_ms": 23,
    "sort_by": "created_at",
    "sort_order": "desc"
  }
}
```

### Descripción de Campos de Respuesta

#### Array `data`

Cada objeto de contacto contiene:

| Campo           | Tipo   | Descripción                                        |
| :-------------- | :----- | :------------------------------------------------- |
| `id`            | string | Identificador único del contacto (UUID)            |
| `name`          | string | Nombre completo del contacto                       |
| `email`         | string | Dirección de email                                 |
| `phone`         | string | Número de teléfono                                 |
| `company`       | string | Nombre de la empresa asociada                      |
| `ref_id`        | string | ID de referencia externa (tu CRM)                  |
| `custom_fields` | object | Campos personalizados (dinámicos por organización) |
| `created_at`    | string | Fecha de creación (ISO 8601)                       |
| `updated_at`    | string | Fecha de última actualización (ISO 8601)           |

#### Objeto `meta`

Información sobre la consulta y paginación:

| Campo               | Tipo    | Descripción                                        |
| :------------------ | :------ | :------------------------------------------------- |
| `page`              | integer | Página actual                                      |
| `page_size`         | integer | Resultados por página                              |
| `total`             | integer | Total de contactos que coinciden con la búsqueda   |
| `execution_time_ms` | integer | Tiempo de ejecución de la consulta en milisegundos |
| `sort_by`           | string  | Campo usado para ordenar                           |
| `sort_order`        | string  | Orden aplicado (`asc`/`desc`)                      |

---

## ⚠️ Manejo de Errores

### Códigos de Error Comunes

| Código HTTP                 | Código de Error           | Descripción                                              |
| :-------------------------- | :------------------------ | :------------------------------------------------------- |
| `400 Bad Request`           | `INVALID_QUERY_PARAMS`    | Parámetros de consulta inválidos.                        |
| `400 Bad Request`           | `INVALID_DATE_FORMAT`     | Formato de fecha incorrecto.                             |
| `400 Bad Request`           | `MISSING_REQUIRED_PARAMS` | Faltan parámetros obligatorios.                          |
| `403 Forbidden`             | `UNAUTHORIZED_ACCESS`     | Acceso no autorizado (API Key o OrgUnit ID incorrectos). |
| `500 Internal Server Error` | `INTERNAL_SERVER_ERROR`   | Error interno del servidor.                              |

### Ejemplos de Respuestas de Error

#### 400 - Parámetros Inválidos

```json
{
  "error": {
    "code": "INVALID_QUERY_PARAMS",
    "message": "Parámetros de consulta inválidos",
    "details": {
      "field": "page_size",
      "error": "page_size debe estar entre 1 y 100"
    }
  }
}
```

#### 400 - Formato de Fecha Incorrecto

```json
{
  "error": {
    "code": "INVALID_DATE_FORMAT",
    "message": "Formato de fecha inválido",
    "details": {
      "field": "created_after",
      "value": "2024-13-01",
      "error": "mes fuera de rango"
    }
  }
}
```

#### 403 - Acceso No Autorizado

```json
{
  "error": {
    "code": "UNAUTHORIZED_ACCESS",
    "message": "Acceso no autorizado a los datos de contactos",
    "details": {
      "error": "clave API inválida o faltante"
    }
  }
}
```

---

## 💻 Ejemplos de Integración por Lenguaje

### JavaScript / Node.js

#### Función Básica

```javascript
async function buscarContactos(termino, pagina = 1) {
  const url = `https://engine.conversia.ai/api/v1/contacts/search-external?search=${encodeURIComponent(
    termino,
  )}&page=${pagina}`;

  try {
    const response = await fetch(url, {
      method: "GET",
      headers: {
        "X-API-KEY": "01989b2c-48d2-7a9e-8b89-14e4a6377dee",
        "X-API-ORGUNIT-ID": "31f1d8e5-9c9e-409d-8336-cf7da99921bb",
      },
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(`Error ${response.status}: ${error.error.message}`);
    }

    return await response.json();
  } catch (error) {
    console.error("Error al buscar contactos:", error);
    throw error;
  }
}

// Uso
buscarContactos("María", 1)
  .then((resultado) => {
    console.log(`Encontrados ${resultado.meta.total} contactos`);
    resultado.data.forEach((contacto) => {
      console.log(`- ${contacto.name} (${contacto.email})`);
    });
  })
  .catch((error) => console.error(error));
```

#### Búsqueda Avanzada con Campos Personalizados

```javascript
async function buscarContactosAvanzado(filtros) {
  const params = new URLSearchParams();

  // Agregar filtros básicos
  if (filtros.search) params.append("search", filtros.search);
  if (filtros.email) params.append("email", filtros.email);
  if (filtros.company) params.append("company", filtros.company);

  // Agregar campos personalizados
  if (filtros.customFields) {
    Object.entries(filtros.customFields).forEach(([key, value]) => {
      params.append(`cf.${key}`, value);
    });
  }

  // Paginación y ordenamiento
  params.append("page", filtros.page || 1);
  params.append("page_size", filtros.pageSize || 20);
  if (filtros.sortBy) params.append("sort_by", filtros.sortBy);
  if (filtros.sortOrder) params.append("sort_order", filtros.sortOrder);

  const url = `https://engine.conversia.ai/api/v1/contacts/search-external?${params}`;

  const response = await fetch(url, {
    headers: {
      "X-API-KEY": process.env.CONVERSIA_API_KEY,
      "X-API-ORGUNIT-ID": process.env.CONVERSIA_ORG_UNIT_ID,
    },
  });

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }

  return await response.json();
}

// Ejemplo de uso avanzado
const filtros = {
  search: "García",
  customFields: {
    estado: "activo",
    region: "EMEA",
    puntuacion__gte: "80",
  },
  sortBy: "updated_at",
  sortOrder: "desc",
  pageSize: 50,
};

buscarContactosAvanzado(filtros)
  .then((resultado) => console.log(resultado))
  .catch((error) => console.error(error));
```

### Python

#### Clase para Manejo de la API

```python
import requests
from typing import Dict, List, Optional

class ConversiaCRM:
    def __init__(self, api_key: str, org_unit_id: str):
        self.api_key = api_key
        self.org_unit_id = org_unit_id
        self.base_url = "https://engine.conversia.ai"

    def _get_headers(self) -> Dict[str, str]:
        return {
            'X-API-KEY': self.api_key,
            'X-API-ORGUNIT-ID': self.org_unit_id
        }

    def buscar_contactos(
        self,
        search: Optional[str] = None,
        email: Optional[str] = None,
        company: Optional[str] = None,
        custom_fields: Optional[Dict[str, str]] = None,
        page: int = 1,
        page_size: int = 20,
        sort_by: Optional[str] = None,
        sort_order: Optional[str] = None
    ) -> Dict:
        """
        Busca contactos en Conversia CRM
        """
        params = {}

        # Parámetros básicos
        if search: params['search'] = search
        if email: params['email'] = email
        if company: params['company'] = company

        # Campos personalizados
        if custom_fields:
            for key, value in custom_fields.items():
                params[f'cf.{key}'] = value

        # Paginación y ordenamiento
        params['page'] = page
        params['page_size'] = page_size
        if sort_by: params['sort_by'] = sort_by
        if sort_order: params['sort_order'] = sort_order

        url = f"{self.base_url}/api/v1/contacts/search-external"

        try:
            response = requests.get(
                url,
                headers=self._get_headers(),
                params=params,
                timeout=30
            )
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            print(f"Error en la solicitud: {e}")
            if hasattr(e, 'response') and e.response is not None:
                try:
                    print(f"Detalle del error: {e.response.json()}")
                except:
                    print(f"Respuesta del servidor: {e.response.text}")
            raise

# Ejemplo de uso
if __name__ == "__main__":
    crm = ConversiaCRM(
        api_key="01989b2c-48d2-7a9e-8b89-14e4a6377dee",
        org_unit_id="31f1d8e5-9c9e-409d-8336-cf7da99921bb"
    )

    # Búsqueda básica
    resultado = crm.buscar_contactos(search="María")
    print(f"Encontrados {resultado['meta']['total']} contactos")

    # Búsqueda avanzada
    resultado_avanzado = crm.buscar_contactos(
        company="Tech",
        custom_fields={
            "estado": "activo",
            "puntuacion__gte": "85"
        },
        sort_by="updated_at",
        sort_order="desc",
        page_size=50
    )

    for contacto in resultado_avanzado['data']:
        print(f"- {contacto['name']} ({contacto['email']}) - {contacto['company']}")
```

### PHP

```php
<?php

class ConversiaCRM {
    private $apiKey;
    private $orgUnitId;
    private $baseUrl = 'https://engine.conversia.ai';

    public function __construct($apiKey, $orgUnitId) {
        $this->apiKey = $apiKey;
        $this->orgUnitId = $orgUnitId;
    }

    private function getHeaders() {
        return [
            'X-API-KEY: ' . $this->apiKey,
            'X-API-ORGUNIT-ID: ' . $this->orgUnitId
        ];
    }

    public function buscarContactos($filtros = []) {
        $params = [];

        // Parámetros básicos
        if (isset($filtros['search'])) $params['search'] = $filtros['search'];
        if (isset($filtros['email'])) $params['email'] = $filtros['email'];
        if (isset($filtros['company'])) $params['company'] = $filtros['company'];

        // Campos personalizados
        if (isset($filtros['custom_fields'])) {
            foreach ($filtros['custom_fields'] as $key => $value) {
                $params["cf.{$key}"] = $value;
            }
        }

        // Paginación y Ordenamiento
        $params['page'] = $filtros['page'] ?? 1;
        $params['page_size'] = $filtros['page_size'] ?? 20;
        if (isset($filtros['sort_by'])) $params['sort_by'] = $filtros['sort_by'];
        if (isset($filtros['sort_order'])) $params['sort_order'] = $filtros['sort_order'];

        $url = $this->baseUrl . '/api/v1/contacts/search-external?' . http_build_query($params);

        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_HTTPHEADER, $this->getHeaders());
        curl_setopt($ch, CURLOPT_TIMEOUT, 30);

        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);

        if ($httpCode !== 200) {
            throw new Exception("Error HTTP {$httpCode}: {$response}");
        }

        return json_decode($response, true);
    }
}

// Ejemplo de uso
try {
    $crm = new ConversiaCRM(
        '01989b2c-48d2-7a9e-8b89-14e4a6377dee',
        '31f1d8e5-9c9e-409d-8336-cf7da99921bb'
    );

    $resultado = $crm->buscarContactos([
        'search' => 'García',
        'custom_fields' => [
            'estado' => 'activo',
            'region' => 'EMEA'
        ],
        'sort_by' => 'name',
        'sort_order' => 'asc',
        'page_size' => 25
    ]);

    echo "Encontrados {$resultado['meta']['total']} contactos\n";
    foreach ($resultado['data'] as $contacto) {
        echo "- {$contacto['name']} ({$contacto['email']})\n";
    }

} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . "\n";
}
?>
```

### cURL (Línea de Comandos)

#### Búsqueda Básica

```bash
curl -X GET "https://engine.conversia.ai/api/v1/contacts/search-external?search=María&page=1&page_size=20" \
 -H "X-API-KEY: 01989b2c-48d2-7a9e-8b89-14e4a6377dee" \
 -H "X-API-ORGUNIT-ID: 31f1d8e5-9c9e-409d-8336-cf7da99921bb"
```

#### Búsqueda Avanzada con Campos Personalizados

```bash
curl -G "https://engine.conversia.ai/api/v1/contacts/search-external" \
 --data-urlencode "cf.estado=activo" \
 --data-urlencode "cf.region=EMEA" \
 --data-urlencode "cf.puntuacion__gte=85" \
 --data-urlencode "sort_by=updated_at" \
 --data-urlencode "sort_order=desc" \
 -H "X-API-KEY: 01989b2c-48d2-7a9e-8b89-14e4a6377dee" \
 -H "X-API-ORGUNIT-ID: 31f1d8e5-9c9e-409d-8336-cf7da99921bb"
```

#### Con Filtros de Fecha

```bash
curl -G "https://engine.conversia.ai/api/v1/contacts/search-external" \
 --data-urlencode "created_after=2024-01-01T00:00:00Z" \
 --data-urlencode "created_before=2024-12-31T23:59:59Z" \
 --data-urlencode "sort_by=created_at" \
 --data-urlencode "sort_order=desc" \
 -H "X-API-KEY: 01989b2c-48d2-7a9e-8b89-14e4a6377dee" \
 -H "X-API-ORGUNIT-ID: 31f1d8e5-9c9e-409d-8336-cf7da99921bb"
```

---

## 🎯 Casos de Uso Comunes

### 1. Sincronización de CRM

**Objetivo:** Sincronizar contactos actualizados recientemente.

```javascript
// Obtener contactos actualizados en las últimas 24 horas
const hace24Horas = new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString();

const contactosRecientes = await buscarContactosAvanzado({
  updatedAfter: hace24Horas,
  sortBy: "updated_at",
  sortOrder: "desc",
  pageSize: 100,
});
```

### 2. Segmentación de Marketing

**Objetivo:** Encontrar contactos VIP para una campaña especial.

```python
# Contactos VIP con alta puntuación
resultado = crm.buscar_contactos(
    custom_fields={
        "etiquetas__in": "vip,premium,oro",
        "puntuacion__gte": "90",
        "estado": "activo"
    },
    sort_by="puntuacion",
    sort_order="desc"
)
```

### 3. Limpieza de Datos

**Objetivo:** Encontrar contactos que requieren revisión (p. ej., sin empresa asignada).

```bash
# Contactos con el campo personalizado 'necesita_revision' y sin empresa
curl -G "https://engine.conversia.ai/api/v1/contacts/search-external" \
 --data-urlencode "company=" \
 --data-urlencode "cf.necesita_revision__exists" \
 -H "X-API-KEY: tu-api-key" \
 -H "X-API-ORGUNIT-ID: tu-org-unit-id"
```

### 4. Reportes por Región

**Objetivo:** Análisis de la cantidad de contactos por región geográfica.

```php
// Obtener el total de contactos por región
$regiones = ['EMEA', 'LATAM', 'APAC', 'NA'];
$reporte = [];

foreach ($regiones as $region) {
    $resultado = $crm->buscarContactos([
        'custom_fields' => ['region' => $region],
        'page_size' => 1  // Solo necesitamos el total del objeto 'meta'
    ]);
    $reporte[$region] = $resultado['meta']['total'];
}
print_r($reporte);
```

---

## ✨ Mejores Prácticas y Consejos

### 🔒 Seguridad

- **Nunca hardcodees las claves API** en tu código fuente.
- **Usa variables de entorno** (`process.env`, `.env` files) para almacenar credenciales de forma segura.
- **Implementa rotación de claves** periódicamente para minimizar riesgos.
- **Monitorea el uso** de tu API key para detectar actividades anómalas.

### ⚡ Rendimiento

- **Usa paginación** siempre, especialmente si esperas más de 100 resultados.
- **Implementa caché** en tu aplicación para consultas frecuentes que no cambian a menudo.
- **Sé específico en tus filtros** en lugar de realizar búsquedas amplias y filtrar del lado del cliente.
- **Monitorea `execution_time_ms`** en las respuestas para identificar consultas lentas.

### 🛠️ Manejo de Errores

- **Implementa reintentos con backoff exponencial** para errores transitorios (como `5xx` o timeouts).
- **Registra los errores** detalladamente para facilitar el debugging.
- **Maneja timeouts** de manera apropiada en tu cliente HTTP.
- **Valida los parámetros** en tu código antes de enviar la solicitud a la API.

---

## 🩺 Solución de Problemas (Troubleshooting)

### ❌ Error `403 - Unauthorized Access`

- **Síntomas:** Recibes el mensaje `"Acceso no autorizado a los datos de contactos"`.
- **Soluciones:**
  1.  Verifica que tu `X-API-KEY` sea correcta y no haya expirado.
  2.  Confirma que el `X-API-ORGUNIT-ID` sea el UUID válido para tu organización.
  3.  Asegúrate de que ambos headers (`X-API-KEY` y `X-API-ORGUNIT-ID`) estén presentes en cada solicitud.
  4.  Contacta a tu administrador para verificar que la clave API tiene los permisos necesarios.

### ❌ Error `400 - Invalid Query Params`

- **Síntomas:** Recibes el mensaje `"Parámetros de consulta inválidos"`.
- **Soluciones:**
  1.  Revisa la sintaxis de los campos personalizados: `cf.nombre_campo__operador`.
  2.  Verifica que `page_size` no exceda el límite de 100.
  3.  Confirma que las fechas estén en formato ISO 8601 (`YYYY-MM-DDTHH:MM:SSZ`).
  4.  Asegúrate de que los operadores (`eq`, `gte`, `in`, etc.) sean válidos y estén escritos correctamente.

### ❌ Resultados Vacíos

- **Síntomas:** La respuesta es un `200 OK` con `"data": []`, pero esperabas resultados.
- **Soluciones:**
  1.  Simplifica tu consulta eliminando filtros uno por uno para identificar cuál está causando el problema.
  2.  Verifica que los valores que buscas en los campos personalizados existan y coincidan exactamente (mayúsculas/minúsculas importan, a menos que uses `icontains`).
  3.  Realiza una búsqueda general con `search` para confirmar que hay datos accesibles.
  4.  Confirma que estás usando el `X-API-ORGUNIT-ID` correcto donde residen los contactos.

### ❌ Timeout o Respuesta Lenta

- **Síntomas:** La consulta tarda más de lo esperado o falla por timeout.
- **Soluciones:**
  1.  Reduce el `page_size` a un valor menor (ej. 25 o 50).
  2.  Agrega filtros más específicos para reducir el conjunto de datos que el servidor debe procesar.
  3.  Utiliza rangos de fechas (`created_after`, `updated_after`) para limitar la búsqueda a un período de tiempo más corto.

---

## ⚖️ Límites y Restricciones

### Límites de la API

| Límite                             | Valor          | Descripción                                          |
| :--------------------------------- | :------------- | :--------------------------------------------------- |
| Resultados por página              | 100            | Máximo valor para `page_size`.                       |
| Timeout de consulta                | 30 segundos    | Tiempo máximo de ejecución por solicitud.            |
| Campos personalizados por consulta | 20             | Máximo número de filtros `cf.*` en una sola llamada. |
| Longitud de búsqueda               | 500 caracteres | Longitud máxima para el parámetro `search`.          |

### Rate Limiting

- **Límite por defecto:** 1000 solicitudes por hora por API key.
- **Headers de respuesta:** Busca los headers `X-RateLimit-*` en la respuesta para conocer tu estado actual (límite, restante, tiempo de reseteo).
- **Recomendación:** Si te acercas al límite, implementa un mecanismo de backoff exponencial en tu código.
