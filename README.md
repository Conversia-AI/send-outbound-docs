### **Documentación de API: Contactos y Mensajería**

Esta documentación detalla los endpoints para crear contactos y enviar mensajes outbound, junto con un flujo de trabajo completo.

**Resumen del Flujo:**
1.  **Crear un Contacto:** Envía una petición `POST` a `/api/v1/contacts` para registrar un nuevo contacto. Puedes proporcionar tu propio `ref_id` o dejar que el sistema genere uno.
2.  **Obtener `ref_id`:** De la respuesta de creación, extrae el `ref_id` del contacto.
3.  **Enviar Mensaje Outbound:** Envía una petición `POST` a `/api/v1/outbound/send`, usando el `integration_id` de tu canal (ej. WhatsApp) y el `ref_id` del contacto como `contact_id`.

---

### **1. Crear Contacto (`POST /api/v1/contacts`)**

Este endpoint permite crear un nuevo contacto en una unidad organizacional específica.

**Endpoint**
`POST /api/v1/contacts`

**Autenticación**
-   **Header (Requerido):** `X-API-KEY: <tu_api_key>`
-   **Unidad Organizacional (org unit):**
    -   **Header (Preferente):** `X-API-ORGUNIT-ID: <uuid>`
    -   **Body (Alternativa):** `org_unit_id: <uuid>`

**Headers**
-   `X-API-KEY: string`
-   `X-API-ORGUNIT-ID: uuid` (Opcional si se envía en el body)
-   `Content-Type: application/json`

**Body (Schema)**
-   `name`: `string` (Requerido)
-   `email`: `string` (Opcional, debe ser un email válido)
-   `phone`: `string` (Opcional, se recomienda formato internacional con código de país)
-   `company`: `string` (Opcional)
-   `ref_id`: `string` (Opcional, identificador único alfanumérico con `-_`. Si no se envía, se puede autogenerar)
-   `auto_generate_ref_id`: `boolean` (Opcional. Si es `true`, el sistema genera un `ref_id` numérico único y de alta precisión, ignorando cualquier `ref_id` enviado)
-   `custom_fields`: `object` (Opcional, hasta 100 claves personalizadas)
-   `org_unit_id`: `uuid` (Opcional si se proveyó en el header)

**Notas de Validación:**
-   `name` es obligatorio y no puede contener solo espacios.
-   `ref_id`, si se envía, debe ser único por `org_unit_id` para poder localizar el contacto posteriormente.
-   `email` y `phone` son validados y normalizados internamente.

#### **Ejemplo de Request (HTTP)**
```http
POST https://tu-dominio/api/v1/contacts
X-API-KEY: {{API_KEY}}
X-API-ORGUNIT-ID: 11111111-2222-3333-4444-555555555555
Content-Type: application/json

{
  "name": "María Rodríguez",
  "email": "maria@example.com",
  "phone": "+54 9 11 1234 5678",
  "company": "Acme SA",
  "auto_generate_ref_id": true,
  "custom_fields": {
    "source": "landing",
    "campaign": "black_friday"
  }
}
```

#### **Ejemplo en cURL**
```bash
curl -X POST 'https://tu-dominio/api/v1/contacts' \
--header 'X-API-KEY: tu_api_key_aqui' \
--header 'X-API-ORGUNIT-ID: 11111111-2222-3333-4444-555555555555' \
--header 'Content-Type: application/json' \
--data-raw '{
  "name": "María Rodríguez",
  "email": "maria@example.com",
  "phone": "+54 9 11 1234 5678",
  "company": "Acme SA",
  "auto_generate_ref_id": true,
  "custom_fields": {
    "source": "landing",
    "campaign": "black_friday"
  }
}'
```

#### **Respuesta Exitosa (201 Created)**
```json
{
  "id": "c0f8b0f1-7d1c-4f3a-9f42-21b8a6f1e9a1",
  "name": "María Rodríguez",
  "email": "maria@example.com",
  "phone": "5491112345678",
  "company": "Acme SA",
  "ref_id": "1723489023456789",
  "custom_fields": {
    "source": "landing",
    "campaign": "black_friday"
  },
  "org_unit_id": "11111111-2222-3333-4444-555555555555",
  "organization_id": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
  "created_at": "2025-08-12T12:34:56Z",
  "updated_at": "2025-08-12T12:34:56Z"
}
```

---

### **2. Enviar Mensaje Outbound (`POST /api/v1/outbound/send`)**

Este endpoint envía un mensaje a un contacto (por ejemplo, una plantilla de WhatsApp) a través de una integración específica.

**Endpoint**
`POST /api/v1/outbound/send`

**Autenticación**
-   **Header (Requerido):** `X-API-KEY: <tu_api_key>`

**Headers**
-   `X-API-KEY: string`
-   `Content-Type: application/json`

**Body (Schema)**
-   `integration_id`: `string` (Requerido. ID de la integración configurada, ej. WhatsApp).
-   `contact_id`: `string | number` (Requerido. **Debe ser el `ref_id` del contacto**).
-   Uso de las Variables para la plantilla del mensaje.
    -   **(Ordenado con prefijo, recomendado, el sufijo depende de la plantilla):** `"data.name:1": "Juan", "data.order_id:2": "ABC-123"`

**Importante:**
-   El sistema buscará al contacto por su `ref_id` (`contact_id`) dentro de la misma `org_unit_id` a la que pertenece la `integration_id`.
-   No mezcles los formatos de `data` en una misma petición.

#### **Ejemplo de Request (HTTP)**
```http
POST https://tu-dominio/api/v1/outbound/send
X-API-KEY: {{API_KEY}}
Content-Type: application/json

{
  "integration_id": "3f6c1b5e-2a4d-4a89-9c77-1b2c3d4e5f60",
  "contact_id": "1723489023456789",
  "data.full_name:1": "María",
  "data.order_code:2": "ORD-7788"
}
```

#### **Ejemplo en cURL**
```bash
curl -X POST 'https://tu-dominio/api/v1/outbound/send' \
--header 'X-API-KEY: tu_api_key_aqui' \
--header 'Content-Type: application/json' \
--data-raw '{
  "integration_id": "3f6c1b5e-2a4d-4a89-9c77-1b2c3d4e5f60",
  "contact_id": "1723489023456789",
  "data.full_name:1": "María",
  "data.order_code:2": "ORD-7788"
}'
```

#### **Respuesta Exitosa (200 OK)**
```json
{
  "success": true,
  "message": "Outbound message sent successfully",
  "integration_id": "3f6c1b5e-2a4d-4a89-9c77-1b2c3d4e5f60",
  "contact_id": "1723489023456789"
}
```

---

### **3. Ejemplo de Implementación en TypeScript**

A continuación se muestra un ejemplo completo en TypeScript que utiliza `axios` para realizar el flujo de crear un contacto y luego enviarle un mensaje outbound.

**Pre-requisitos:**
-   Node.js y npm/yarn instalados.
-   Instalar las dependencias: `npm install axios typescript ts-node @types/node`

**Código (`run-flow.ts`)**
```typescript
import axios, { AxiosInstance } from "axios";

// --- Configuración ---
const API_BASE_URL = "https://tu-dominio/api/v1";
const API_KEY = process.env.API_KEY || "tu_api_key_aqui";
const ORG_UNIT_ID = "11111111-2222-3333-4444-555555555555";
const WHATSAPP_INTEGRATION_ID = "3f6c1b5e-2a4d-4a89-9c77-1b2c3d4e5f60";

// --- Definición de Tipos (Interfaces) ---
interface CustomFields {
  [key: string]: any;
}

interface CreateContactPayload {
  name: string;
  phone: string;
  email?: string;
  ref_id?: string;
  auto_generate_ref_id?: boolean;
  custom_fields?: CustomFields;
}

interface ContactResponse {
  id: string;
  ref_id: string;
  name: string;
  phone: string;
  [key: string]: any;
}

interface SendOutboundPayload {
  integration_id: string;
  contact_id: string | number; // Este es el ref_id del contacto
  data?: { [key: string]: any };
}

interface SendOutboundResponse {
  success: boolean;
  message: string;
  integration_id: string;
  contact_id: string | number;
}

// --- Cliente de API ---
const apiClient: AxiosInstance = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    "Content-Type": "application/json",
    "X-API-KEY": API_KEY,
  },
});

// --- Flujo Principal ---
async function main() {
  console.log("🚀 Iniciando flujo: Crear contacto y enviar outbound...");

  try {
    // --- Paso 1: Crear el Contacto ---
    const contactPayload: CreateContactPayload = {
      name: "Juan Pérez",
      phone: "+52 1 55 9876 5432",
      email: "juan.perez@example.com",
      auto_generate_ref_id: true, // Dejamos que el sistema genere el ref_id
      custom_fields: {
        lead_source: "api_example",
      },
    };

    console.log("\n1. Creando contacto...");
    const { data: newContact } = await apiClient.post<ContactResponse>(
      "/contacts",
      contactPayload,
      { headers: { "X-API-ORGUNIT-ID": ORG_UNIT_ID } }
    );

    console.log("✅ Contacto creado con éxito!");
    console.log(`   - ID Interno: ${newContact.id}`);
    console.log(`   - Ref ID: ${newContact.ref_id}`);

    const contactRefId = newContact.ref_id;

    // --- Paso 2: Enviar Mensaje Outbound ---
    const outboundPayload: SendOutboundPayload = {
      integration_id: WHATSAPP_INTEGRATION_ID,
      contact_id: contactRefId, // Usamos el ref_id obtenido
        // Usando el formato recomendado acorde al template
        "data.nombre_cliente:1": newContact.name.split(" ")[0], // "Juan"
        "data.numero_ticket:2": `TICKET-${Math.floor(Math.random() * 10000)}`
    };

    console.log(`\n2. Enviando mensaje outbound al contacto ${contactRefId}...`);
    const { data: outboundResponse } =
      await apiClient.post<SendOutboundResponse>(
        "/outbound/send",
        outboundPayload
      );

    if (outboundResponse.success) {
      console.log("✅ Mensaje enviado con éxito!");
      console.log(`   - Mensaje: ${outboundResponse.message}`);
    } else {
      console.error("❌ Error al enviar el mensaje:", outboundResponse);
    }
  } catch (error) {
    console.error("\n❌ Ocurrió un error durante el flujo:");
    if (axios.isAxiosError(error)) {
      console.error(`   - Status: ${error.response?.status}`);
      console.error(`   - Data: ${JSON.stringify(error.response?.data)}`);
    } else {
      console.error(error);
    }
  }
}

// Ejecutar el flujo
main();
```
