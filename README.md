# 🌐 Comunicación en Redes — Actividad Grupal

Guía sobre cómo viajan los datos entre aplicaciones y cómo impacta directamente en el desarrollo: APIs, frontend-backend, puertos y debugging.

> *"Cada vez que haces un fetch, abres Postman o levantas un backend… estás usando protocolos, puertos y reglas de comunicación. Si no entiendes esto, no sabes realmente cómo funciona tu aplicación."*

---

## 🎯 Objetivo

- Comprender cómo viajan los datos entre aplicaciones
- Entender la diferencia entre TCP y UDP
- Saber qué es un puerto y por qué importa
- Conocer cómo funciona HTTP y una petición completa

---

## 1. Conceptos base

| Concepto | Definición simple | Nivel técnico breve | Ejemplo real |
|---|---|---|---|
| **TCP** | Protocolo que garantiza que los datos lleguen completos y en orden. | Capa 4 del OSI. Establece conexión antes de enviar. Confirma cada entrega. | Cargar una página web, hacer un fetch a una API. |
| **UDP** | Protocolo más rápido que TCP pero sin garantía de entrega. | Capa 4 del OSI. Envía datos sin establecer conexión ni confirmar recepción. | Videollamadas, streaming, juegos en línea. |
| **Puerto** | Número que identifica un servicio específico dentro de un servidor. | Valor de 0 a 65535. La IP lleva al servidor, el puerto lleva al servicio correcto. | Puerto `3000` = tu app local. Puerto `443` = HTTPS. |
| **HTTP** | Protocolo que define cómo se comunican el cliente y el servidor. | Capa 7 del OSI. Basado en peticiones y respuestas. Usa verbos (GET, POST, PUT, DELETE). | Cada vez que abres una web o haces un `fetch` usas HTTP. |

---

## 2. TCP vs UDP

- **Principal diferencia:** TCP establece una conexión antes de enviar datos y confirma que cada paquete llegó. UDP dispara los datos sin esperar confirmación.
- **¿Cuál es más confiable?** TCP. Garantiza que los datos llegan completos y en orden. Si un paquete se pierde, lo reenvía.
- **¿Cuál es más rápido?** UDP. Al no confirmar cada entrega ni reenviar paquetes perdidos, es mucho más veloz. En streaming, es mejor perder un frame que esperar.

| Protocolo | Característica clave | Uso típico |
|---|---|---|
| **TCP** | Conexión establecida, entrega garantizada, orden garantizado. | APIs REST, carga de páginas web, emails, descarga de archivos. |
| **UDP** | Sin conexión, sin confirmación, más rápido. | Videollamadas, streaming, juegos online, DNS. |

---

## 3. Puertos

Un puerto es un número que identifica un servicio específico dentro de un servidor. Cuando un paquete llega a una IP, el puerto le indica a qué aplicación entregarlo. En un mismo servidor pueden correr múltiples servicios: sin puertos, el sistema no sabría a cuál entregarle cada paquete.

| Puerto | Servicio |
|---|---|
| `80` | HTTP — tráfico web sin cifrar |
| `443` | HTTPS — tráfico web cifrado |
| `3000` | Convención para apps Node.js en desarrollo local |
| `5432` | PostgreSQL — base de datos |
| `3306` | MySQL — base de datos |
| `27017` | MongoDB |

### ¿Qué pasa cuando ejecutas `npm run dev`?
Tu app levanta un servidor que empieza a escuchar peticiones en un puerto específico (generalmente `3000`). Cualquier petición que llegue a `localhost:3000` es recibida por tu app. Si ese puerto ya está en uso, el servidor no puede arrancar y obtienes el error `Port already in use`.

---

## 4. HTTP — HyperText Transfer Protocol

HTTP es el protocolo que define cómo se comunican el cliente (navegador o frontend) y el servidor (backend). Funciona con un sistema de petición y respuesta: el cliente pide algo, el servidor responde.

### Verbos HTTP

| Verbo | ¿Qué hace? | Ejemplo |
|---|---|---|
| `GET` | Pide datos al servidor. No modifica nada. | `GET /usuarios` — trae la lista de usuarios. |
| `POST` | Envía datos nuevos al servidor para crear algo. | `POST /usuarios` — crea un nuevo usuario. |
| `PUT` | Envía datos para reemplazar un recurso existente. | `PUT /usuarios/5` — reemplaza el usuario con id 5. |
| `DELETE` | Elimina un recurso del servidor. | `DELETE /usuarios/5` — elimina el usuario con id 5. |

### Flujo de una petición HTTP

```
1. Frontend hace fetch('http://localhost:3000/api/usuarios')
2. Se genera una petición HTTP GET
3. Viaja por la red usando TCP hasta el backend
4. El backend procesa la petición
5. El backend responde con un JSON y un código de estado (ej: 200 OK)
6. El frontend recibe la respuesta y la muestra
```

---

## 5. Conexión directa con desarrollo

### ¿Qué pasa cuando haces `fetch("http://localhost:3000/api")`?

| Aspecto | Detalle |
|---|---|
| **Protocolo** | HTTP (o HTTPS si está configurado con certificado) |
| **Puerto** | `3000` — el backend debe estar escuchando en ese puerto |
| **Transporte** | TCP — la conexión se establece, la petición se envía y la respuesta se recibe |
| **Flujo** | Navegador resuelve `localhost` → conecta a `127.0.0.1:3000` → envía GET → backend responde → frontend recibe JSON |

> Si el backend no está corriendo o está en otro puerto, obtienes `Connection refused`. Si está en otro dominio sin CORS configurado, el navegador bloquea la respuesta aunque el servidor responda.

---

## 6. Debugging — errores comunes

| Error | Qué significa | Qué revisarías |
|---|---|---|
| `Port already in use` | Otro proceso ya usa ese puerto. Tu app no puede arrancar. | Buscar con `lsof -i :3000` y terminar el proceso, o cambiar de puerto. |
| `Connection refused` | El servidor no está escuchando en esa IP y puerto. | ¿Está corriendo el backend? ¿Puerto correcto? ¿Escucha en `0.0.0.0`? |
| `Timeout` | El servidor no respondió en el tiempo esperado. | ¿Hay firewall bloqueando? ¿La red llega al servidor? ¿Servidor saturado? |

---

## 7. Caso práctico — frontend no se conecta al backend

- **¿Problema de puerto?** Sí, muy común. El frontend apunta a `localhost:3000` pero el backend corre en `localhost:4000`. O el puerto estaba ocupado y el backend no arrancó.
- **¿Problema de protocolo?** Sí. El frontend usa `https://` pero el backend solo acepta `http://`. O hay error de CORS porque el protocolo o dominio no coincide.
- **¿Problema de red?** Sí. En producción, el firewall bloquea el puerto o el servidor está caído.

### Checklist de diagnóstico

```
1. ¿El backend está corriendo? → revisa la terminal
2. ¿El puerto en el fetch coincide con el del backend?
3. ¿El protocolo coincide (http vs https)?
4. ¿Hay error de CORS en la consola del navegador?
5. ¿El firewall permite el puerto en producción?
```

---

## 8. Analogías

| Concepto | Analogía |
|---|---|
| **TCP** | Una llamada telefónica: primero se establece la conexión, luego se habla, y ambos confirman que se escuchan. |
| **UDP** | Mandar un mensaje de voz sin saber si la otra persona lo recibió o escuchó. |
| **Puerto** | El número de departamento en un edificio: la IP es la dirección del edificio, el puerto es el depto específico. |
| **HTTP** | El idioma que usan el cliente y el servidor para entenderse: tiene reglas, verbos y respuestas estándar. |

---

## 9. Bonus — HTTPS, REST y Endpoints

### ¿Qué es HTTPS?
Es HTTP con una capa de cifrado (TLS/SSL). Todo lo que viaja entre el cliente y el servidor está encriptado. En producción, toda app debería usar HTTPS: sin él, los datos viajan en texto plano y pueden ser interceptados.

### ¿Qué es REST?
REST (Representational State Transfer) es un estilo de arquitectura para diseñar APIs. Define reglas: usar los verbos HTTP correctamente, que cada URL represente un recurso, y que el servidor no guarde estado entre peticiones.

### ¿Qué es un endpoint?
Es una URL específica de una API que responde a un tipo de petición. Es el punto de entrada a un recurso o acción concreta del servidor.

```
GET    /api/usuarios      → trae todos los usuarios
GET    /api/usuarios/5    → trae el usuario con id 5
POST   /api/usuarios      → crea un nuevo usuario
DELETE /api/usuarios/5    → elimina el usuario con id 5
```

---

## 10. Glosario de términos

| Término | ¿Qué es? (en simple) | Ejemplo de uso |
|---|---|---|
| **TCP** | Protocolo que garantiza que los datos lleguen completos y en orden. | Cada `fetch` a una API usa TCP para asegurar que la respuesta llegue completa. |
| **UDP** | Protocolo más rápido que TCP pero sin garantía de entrega. | Videollamadas usan UDP. Si un frame se pierde, no se reenvía — es mejor seguir. |
| **Puerto** | Número que identifica un servicio específico dentro de un servidor. | `fetch('http://localhost:3000')` → el sistema sabe exactamente a qué app ir. |
| **HTTP** | Protocolo de petición y respuesta entre cliente y servidor. | Cuando abres una web o haces un `fetch`, usas HTTP para pedir y recibir datos. |
| **HTTPS** | HTTP con cifrado. Todo lo que viaja entre cliente y servidor está encriptado. | En producción usar `https://` para que los datos no viajen en texto plano. |
| **GET** | Verbo HTTP para pedir datos al servidor. No modifica nada. | `fetch('/api/usuarios')` usa GET por defecto. Trae la lista sin cambiar nada. |
| **POST** | Verbo HTTP para enviar datos nuevos al servidor y crear algo. | `fetch('/api/usuarios', { method: 'POST', body: ... })` crea un nuevo usuario. |
| **PUT** | Verbo HTTP para reemplazar un recurso existente en el servidor. | `fetch('/api/usuarios/5', { method: 'PUT', body: ... })` reemplaza el usuario 5. |
| **DELETE** | Verbo HTTP para eliminar un recurso del servidor. | `fetch('/api/usuarios/5', { method: 'DELETE' })` elimina el usuario con id 5. |
| **Request** | La petición que el cliente envía al servidor. Incluye verbo, URL y a veces body. | Cuando haces `fetch('/api/datos')`, estás enviando una request al backend. |
| **Response** | La respuesta que el servidor devuelve al cliente tras recibir una request. | El backend responde con `200 OK` y un JSON con los datos solicitados. |
| **Status code** | Número que indica el resultado de una petición HTTP. | `200` = éxito. `404` = no encontrado. `500` = error del servidor. `401` = no autorizado. |
| **REST** | Estilo de arquitectura para diseñar APIs usando HTTP correctamente. | API REST usa GET para leer, POST para crear, PUT para actualizar y DELETE para eliminar. |
| **Endpoint** | URL específica de una API que responde a un tipo de petición. | `GET /api/usuarios` es un endpoint que devuelve todos los usuarios. |
| **CORS** | Mecanismo de seguridad del navegador que bloquea peticiones entre dominios distintos. | Frontend en `localhost:5173` hace fetch a `localhost:3000` → sin CORS configurado, el navegador bloquea. |
| **Header** | Información adicional que viaja junto a una petición o respuesta HTTP. | `Content-Type: application/json` le dice al servidor que el body viene en JSON. |
| **Body** | El contenido principal de una petición o respuesta. Generalmente en JSON. | Al hacer POST, el body contiene: `{ "nombre": "Ana", "email": "ana@mail.com" }`. |
| **JSON** | Formato estándar para intercambiar datos entre frontend y backend. | El backend responde con `{ "id": 1, "nombre": "Ana" }`. El frontend lo muestra. |
| **localhost** | Nombre que apunta siempre a tu propia máquina. Equivale a `127.0.0.1`. | Tu backend corre en `http://localhost:3000`. Solo tú puedes acceder desde tu equipo. |
| **Firewall** | Sistema que filtra el tráfico de red según reglas. Puede bloquear puertos o IPs. | Si el firewall no permite el puerto `3000`, nadie puede conectarse aunque el backend esté corriendo. |
| **TLS/SSL** | Protocolo de cifrado que hace que HTTP se convierta en HTTPS. | Cuando ves el candado en el navegador, la conexión usa TLS y los datos viajan cifrados. |
| **Protocolo** | Conjunto de reglas que define cómo se comunican dos sistemas. | HTTP, TCP y UDP son protocolos. Definen el formato y las reglas de la comunicación. |

---

## 💡 TL;DR

| Concepto | En simple |
|---|---|
| **TCP** | Entrega garantizada y en orden — ideal para APIs y webs |
| **UDP** | Más rápido, sin garantía — ideal para streaming y videollamadas |
| **Puerto** | Identifica un servicio específico dentro de un servidor |
| **HTTP** | Protocolo de petición-respuesta con verbos GET, POST, PUT, DELETE |
| **HTTPS** | HTTP con cifrado — obligatorio en producción |
| **REST** | Estándar de diseño de APIs usando HTTP correctamente |
| **Endpoint** | URL específica que expone una acción o recurso del servidor |

---

> *"Cuando entiendes cómo viajan los datos… dejas de programar a ciegas y empiezas a construir sistemas de verdad."*
