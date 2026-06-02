# Construye tu agente de ventas con IA usando Claude Code
### Guía paso a paso desde cero

Esto no es una guía para descargar un programa ya hecho. Es una guía para que **tú construyas tu propio agente de ventas** con IA, usando Claude Code como tu desarrollador. Tú le das las instrucciones, él escribe el código.

Si sigues esta guía, en uno o dos fines de semana tendrás un agente de WhatsApp que cualifica leads, agenda reuniones y hace seguimiento mientras tú duermes. Y lo más importante: será **tu sistema**, no el de otra persona.

---

## Lo que vas a tener al terminar

Un número de WhatsApp que responde solo. Cuando alguien te escribe:

- El agente saluda y le manda los recursos de tu oferta
- Le hace preguntas para entender su negocio
- Lo puntúa internamente del 0 al 100 según tus criterios
- Si encaja, le propone una llamada y la agenda en tu calendario
- Si no responde, le hace follow-up a las 2h, 24h y 72h
- Tu CRM (GoHighLevel) se actualiza solo

Tú solo recibes a los leads ya cualificados, con todo el contexto de la conversación.

---

## Lo que necesitas reunir antes de empezar

Una hora reuniendo cuentas y se acabó.

| Servicio | Para qué | Coste |
|---|---|---|
| **Anthropic** | El cerebro del agente | desde $5/mes |
| **OpenAI** | Transcribe audios | $1-5/mes |
| **GoHighLevel** | CRM y calendario | $97/mes |
| **Contabo** | El servidor donde vive el agente | desde 7€/mes |
| **Claude Code Pro** | El que escribe el código por ti | $20/mes |
| **Dominio** | Para que WhatsApp llegue al servidor | ~10€/año |

> 💡 **Total mensual:** ~150-180€. Una sola venta cubre 3-6 meses.

**Crea las cuentas:**

1. **Anthropic:** [console.anthropic.com](https://console.anthropic.com) → Sign up → añade $20 de crédito (es para la API que usará el agente)
2. **OpenAI:** [platform.openai.com](https://platform.openai.com) → Sign up → añade $10 de crédito
3. **GoHighLevel:** si no lo tienes ya, contrátalo (lo necesitas sí o sí)
4. **Contabo:** [contabo.com](https://contabo.com) — todavía no contrates nada, lo hacemos en el Paso 2
5. **Claude Code Pro:** [claude.com/code](https://claude.com/code) → suscríbete al plan Pro
6. **Dominio:** Namecheap, Cloudflare o el que prefieras

> ⚠️ **Apunta todo en un gestor de contraseñas.** Vas a manejar muchas claves.

---

## PASO 1 — Saca tus claves de API

Cada servicio te da una llave para que tu agente pueda hablar con él.

### Anthropic (Claude)
1. Entra en [console.anthropic.com](https://console.anthropic.com)
2. Menú izquierdo → **API Keys** → **Create Key**
3. Nómbrala "agente-whatsapp"
4. Copia la clave (empieza por `sk-ant-...`). ⚠️ Solo se ve una vez.

### OpenAI
1. Entra en [platform.openai.com](https://platform.openai.com)
2. Arriba derecha → **API Keys** → **Create new secret key**
3. Copia la clave (empieza por `sk-proj-...`)

### GoHighLevel
1. En tu subaccount → **Settings** → **Business Profile** → apunta el **Location ID**
2. **Settings** → **API Keys** → genera un **Private Integration Token**
3. Copia el token

**✅ Checkpoint:** tienes 3 claves + tu Location ID guardados.

---

## PASO 2 — Contrata tu servidor en Contabo

### Contratar el VPS
1. Entra en [contabo.com](https://contabo.com)
2. **Cloud VPS** → elige **VPS S Cloud** (4 vCPU, 8 GB RAM)
3. Configuración:
   - **Region:** Europe (Alemania)
   - **Image:** Ubuntu 24.04
   - **Storage:** 100 GB SSD
   - **Object storage:** déjalo como está
4. Pon una contraseña fuerte para `root` y apúntala
5. Paga

A los 5-10 minutos te llega un email con la **IP de tu servidor**. Apúntala.

**✅ Checkpoint:** tienes la IP del VPS y la contraseña de root.

---

## PASO 3 — Instala Easypanel en tu servidor

Easypanel es un panel visual que va a manejar todo lo difícil por ti. No vas a tocar Docker a mano.

### Conectarte al servidor
En tu ordenador abre la **Terminal** (Mac) o **PowerShell** (Windows):

```
ssh root@TU_IP
```

Te pide la contraseña de Contabo. Cuando la pongas no verás letras (es normal). Pulsa Enter.

Si ves algo así estás dentro:
```
root@vmi1234567:~#
```

### Instala Easypanel con un comando

```
curl -sSL https://get.easypanel.io | sh
```

Tarda unos minutos. Cuando termine te dirá algo como:
```
Easypanel installed successfully
Visit: http://TU_IP:3000
```

### Configura Easypanel
1. Abre en tu navegador: `http://TU_IP:3000`
2. Crea tu cuenta de admin (email + contraseña)
3. Ya estás dentro del panel

**✅ Checkpoint:** ves el panel de Easypanel en tu navegador.

---

## PASO 4 — Despliega PostgreSQL y Redis

Tu agente necesita una base de datos (Postgres) y una caché (Redis). Easypanel los instala con clics.

### Crear el proyecto
1. En Easypanel → **Create Project** → nómbralo `agente-ventas`
2. Entra en el proyecto

### Añadir PostgreSQL
1. **+ Service** → **Postgres**
2. Nombre: `db`
3. Usuario: `agentes`
4. Genera una contraseña fuerte y apúntala
5. Database: `sistema_agentes`
6. **Create**

### Añadir Redis
1. **+ Service** → **Redis**
2. Nombre: `redis`
3. Genera una contraseña fuerte y apúntala
4. **Create**

**✅ Checkpoint:** ves dos servicios verdes en Easypanel: `db` y `redis`.

---

## PASO 5 — Prepara tu ordenador para Claude Code

Vas a programar... o mejor dicho, **vas a hacer que Claude Code programe por ti**.

### Instala lo necesario

**1. Git** (para guardar tu código):
- Mac: ya viene instalado
- Windows: descarga desde [git-scm.com](https://git-scm.com/download/win) → instala con todas las opciones por defecto

**2. Node.js** (lo necesita Claude Code):
- Descarga desde [nodejs.org](https://nodejs.org) → versión LTS

**3. Visual Studio Code** (tu editor):
- Descarga desde [code.visualstudio.com](https://code.visualstudio.com)

**4. Claude Code:**
- Abre Visual Studio Code
- Pestaña de extensiones (icono de bloques en la barra lateral)
- Busca "Claude Code" → instala la oficial de Anthropic
- Inicia sesión con tu cuenta Pro

### Crea tu cuenta de GitHub
Si no tienes:
1. [github.com](https://github.com) → Sign up
2. Crea un repositorio nuevo llamado `mi-agente-ventas` (privado)
3. No le añadas README ni nada

**✅ Checkpoint:** tienes Claude Code instalado y logueado en VS Code, y un repo vacío en GitHub.

---

## PASO 6 — Crea la carpeta del proyecto

En tu ordenador:

1. Crea una carpeta nueva en tu escritorio: `mi-agente-ventas`
2. Abre VS Code
3. **File** → **Open Folder** → selecciona esa carpeta
4. Abre la terminal integrada de VS Code: `Ctrl + ñ` (Windows) o `Cmd + ñ` (Mac)

Inicializa el proyecto:

```
git init
git remote add origin https://github.com/TU_USUARIO/mi-agente-ventas.git
```

**✅ Checkpoint:** tienes una carpeta vacía abierta en VS Code, conectada a tu repo de GitHub.

---

## PASO 7 — Personaliza el MASTER PROMPT con tu negocio

Esto es lo único que requiere cabeza. Vas a rellenar una plantilla con la info de tu negocio. Esa plantilla es lo que le vas a pasar a Claude Code para que construya el agente.

**Abajo tienes el MASTER PROMPT completo.** Cópialo entero, abre un archivo de texto en cualquier sitio y rellena los `[CORCHETES]` con tu información.

> 💡 **Consejo:** dedica 1-2 horas a rellenar bien los corchetes. Cuanto más detallado y honesto seas, mejor hablará tu agente. No es un trámite, es la receta de tu vendedor.

---

## 📋 MASTER PROMPT (cópialo entero)

```
Quiero que me construyas un agente de ventas de WhatsApp con IA. Trabajaremos
módulo a módulo, con commits pequeños y descriptivos. Pregúntame cuando tengas
dudas en lugar de asumir.

═══════════════════════════════════════════════════════════════
QUÉ HACE EL AGENTE
═══════════════════════════════════════════════════════════════

- Recibe mensajes de WhatsApp y responde con un agente conversacional en mi tono
- Soporta texto, audio (transcribe con Whisper) e imagen (Claude Vision)
- Mantiene memoria persistente: si un lead vuelve días después, recuerda
- Implementa una máquina de estados con estas fases:
    nuevo → apertura_recursos → descubrimiento → cualificacion →
    propuesta_llamada → agendado / descartado / follow_up
    (con saltos a reencuadre y manejo_objecion cuando aplique)
- Puntúa al lead del 0 al 100 según mi ICP en cada turno
- Agenda, reagenda y cancela citas en GHL Calendar
- Crea contactos en GHL con tags y notas de la conversación
- Mueve el lead por las etapas del pipeline de GHL automáticamente
- Hace follow-up automático configurable (por defecto 2h, 24h, 72h)
- Pasa a humano si detecta confusión grave o el lead lo pide
- Comandos de admin desde mi WhatsApp: /reset {phone}, /help

═══════════════════════════════════════════════════════════════
STACK TÉCNICO
═══════════════════════════════════════════════════════════════

- Python 3.11 + FastAPI (backend)
- PostgreSQL (memoria persistente del agente)
- Redis (buffer de mensajes con ventana 3 segundos)
- Anthropic Claude (cerebro del agente, modelo Sonnet 4.6)
- OpenAI Whisper (transcripción de audios)
- Evolution API (WhatsApp en desarrollo)
- Adaptador para Meta Cloud API (preparado para producción)
- GoHighLevel API (CRM)
- Chatwoot (opcional, supervisión humana)
- SQLAlchemy + Alembic (modelos y migraciones)
- pydantic-settings (configuración por .env)

═══════════════════════════════════════════════════════════════
ESTRUCTURA DEL PROYECTO
═══════════════════════════════════════════════════════════════

mi-agente-ventas/
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── database.py
│   │   ├── models/         (Lead, Conversation, Message)
│   │   ├── webhooks/       (whatsapp, ghl, chatwoot)
│   │   ├── agents/sales/   (agente conversacional)
│   │   ├── integrations/   (anthropic, whisper, ghl, evolution)
│   │   ├── services/       (followup, message_buffer)
│   │   └── utils/
│   ├── alembic/            (migraciones)
│   ├── tests/
│   ├── requirements.txt
│   └── Dockerfile
├── docs/                   (documentación generada por ti)
├── .env.example
├── .gitignore
└── docker-compose.yml      (solo para desarrollo local)

═══════════════════════════════════════════════════════════════
DESPLIEGUE
═══════════════════════════════════════════════════════════════

Voy a desplegar en Easypanel sobre un VPS de Contabo. Prepárame:
- Dockerfile optimizado para producción
- .env.example con TODAS las variables documentadas
- Logs estructurados (formato legible en Easypanel)
- README con instrucciones de despliegue claras

═══════════════════════════════════════════════════════════════
MI NEGOCIO
═══════════════════════════════════════════════════════════════

[DESCRIBE_AQUÍ_TU_NEGOCIO_EN_3-5_FRASES.
EJEMPLO: Mentoría estratégica de 3 meses para agencias pequeñas de IA
que tienen problemas para conseguir sus primeros clientes recurrentes.
Trabajo 1-a-1 enseñando un sistema de ventas y un sistema de delivery
que les permite escalar sin contratar.]

═══════════════════════════════════════════════════════════════
MI OFERTA
═══════════════════════════════════════════════════════════════

Nombre: [NOMBRE_DE_TU_OFERTA]
Precio: [PRECIO_EN_EUROS]
Duración: [SEMANAS_O_MESES]
Qué incluye:
- [BULLET_1]
- [BULLET_2]
- [BULLET_3]

Resultado prometido al cliente: [RESULTADO_CONCRETO]

═══════════════════════════════════════════════════════════════
MI AVATAR (a quién va dirigido)
═══════════════════════════════════════════════════════════════

[DESCRIBE_TU_CLIENTE_IDEAL.
EJEMPLO: Dueños de agencias pequeñas de IA o automatización
(1-10 personas) en España y LATAM, con menos de 6 meses de
recorrido, que están luchando para conseguir sus primeros
3-5 clientes y todavía no tienen un sistema comercial funcionando.]

═══════════════════════════════════════════════════════════════
MI TONO DE VOZ
═══════════════════════════════════════════════════════════════

Características:
- [EJ: Minúsculas siempre, sin tildes en "que/como/donde"]
- [EJ: Frases cortas, directas, sin floreo]
- [EJ: Uso máximo 1 emoji por mensaje, casi nunca]
- [EJ: Tono de mentor con criterio, no de vendedor presionando]

Palabras prohibidas: [LISTA]
Palabras obligatorias: [LISTA]

Ejemplos REALES de cómo hablo yo (pega 5-10 mensajes que hayas
escrito tú a clientes):
- [MENSAJE_REAL_1]
- [MENSAJE_REAL_2]
- [MENSAJE_REAL_3]
- ...

═══════════════════════════════════════════════════════════════
PREGUNTAS DE CUALIFICACIÓN
═══════════════════════════════════════════════════════════════

Estas son las preguntas que el agente debe hacer durante el
descubrimiento, una a una, con naturalidad:

1. [PREGUNTA_1]
2. [PREGUNTA_2]
3. [PREGUNTA_3]
4. [PREGUNTA_4]
5. [PREGUNTA_5]

Criterios de scoring (0-100):
- [CRITERIO_1] → [PESO]
- [CRITERIO_2] → [PESO]
- [CRITERIO_3] → [PESO]

Score mínimo para proponer llamada: [55 por defecto]

═══════════════════════════════════════════════════════════════
OBJECIONES TÍPICAS Y CÓMO LAS MANEJO
═══════════════════════════════════════════════════════════════

Objeción: "es caro"
Respuesta: [TU_RESPUESTA_REAL]

Objeción: "no tengo tiempo"
Respuesta: [TU_RESPUESTA_REAL]

Objeción: "lo voy a pensar"
Respuesta: [TU_RESPUESTA_REAL]

Objeción: [OTRA_QUE_TE_HAGAN_MUCHO]
Respuesta: [TU_RESPUESTA_REAL]

═══════════════════════════════════════════════════════════════
ESTRUCTURA DE FOLLOW-UP
═══════════════════════════════════════════════════════════════

FU1 a las 2 horas: tono casual, comprueba que recibió los recursos.
                   No menciona la llamada todavía.

FU2 a las 24 horas: añade un valor o insight relevante. Introduce
                    la idea de la llamada de forma suave.

FU3 a las 72 horas: mensaje de cierre sin presión. "si no es el
                    momento, sin problema". Deja la puerta abierta.

═══════════════════════════════════════════════════════════════
INSTRUCCIONES DE TRABAJO
═══════════════════════════════════════════════════════════════

Construye el sistema en este orden, módulo a módulo:

1. Estructura de carpetas + .gitignore + .env.example
2. config.py (settings con pydantic) + database.py (SQLAlchemy async)
3. Modelos: Lead, Conversation, Message + primera migración Alembic
4. Webhook de WhatsApp (Evolution) + integración con Anthropic
5. Agente de ventas v1 (sin memoria, sin state machine, solo conversación)
6. Memoria persistente + state machine de fases + scoring
7. Buffer de mensajes en Redis (ventana 3 segundos)
8. Whisper para audios + Claude Vision para imágenes
9. Integración con GHL (contactos, tags, pipeline, calendario)
10. Sistema de follow-ups automáticos
11. Comandos de admin desde WhatsApp
12. Dockerfile + documentación final

Después de cada módulo:
- Haz un commit descriptivo
- Actualiza /docs con lo que has añadido
- Pídeme que pruebe antes de seguir al siguiente módulo

EMPIEZA POR EL PASO 1.
```

---

## PASO 8 — Pega el MASTER PROMPT en Claude Code

1. En VS Code, abre el panel de Claude Code (icono en la barra lateral)
2. Pega el master prompt completo (con tus corchetes ya rellenados) en el chat
3. Pulsa Enter

**Claude Code va a empezar a construir.** Te irá preguntando cosas y mostrándote los archivos que va creando. Tu trabajo aquí es:

- Leer lo que hace y decir "sí" o "no" cuando pregunte
- Confirmar comandos antes de que los ejecute
- Probar cada módulo cuando te pida

> 💡 **Tip:** no le pidas que vaya rápido. Si va módulo a módulo, lo entenderás. Si va todo de golpe, solo tendrás un código que no controlas.

> ⚠️ **Si Claude Code se queda sin contexto** (te dirá algo como "context too long"), abre una nueva conversación y dile: "continúa desde donde estábamos, mira el código que ya has escrito".

**✅ Checkpoint:** tienes los primeros archivos creados (carpetas, `.gitignore`, `.env.example`).

---

## PASO 9 — Sube tu código a GitHub

Después de cada módulo, guarda los cambios:

```
git add .
git commit -m "modulo X completado"
git push
```

Si nunca has hecho `git push` antes, GitHub te pedirá usuario y un **Personal Access Token**. Lo generas en GitHub → Settings → Developer Settings → Personal Access Tokens → genera uno con permisos de `repo`.

**✅ Checkpoint:** ves tu código en GitHub.

---

## PASO 10 — Despliega el backend en Easypanel

Cuando Claude Code te haya construido al menos los primeros módulos (estructura + agente conversacional básico), ya puedes desplegar.

### Desde Easypanel
1. Vuelve a tu proyecto `agente-ventas` en Easypanel
2. **+ Service** → **App**
3. Configuración:
   - **Source:** GitHub
   - **Repository:** tu repo `mi-agente-ventas`
   - **Branch:** `main`
   - **Build Path:** `/backend`
4. **Environment Variables:** copia el contenido de tu `.env` aquí
5. **Deploy**

Easypanel construye y arranca tu app. Ve a la pestaña **Logs** para ver lo que hace.

Si arranca bien verás:
```
sistema-agentes arrancando (env=production)
Uvicorn running on http://0.0.0.0:8000
```

**✅ Checkpoint:** tu app corre en Easypanel.

---

## PASO 11 — Conecta WhatsApp con Evolution API

Evolution API también se despliega desde Easypanel.

1. En el mismo proyecto → **+ Service** → **App**
2. **Source:** Docker Image
3. **Image:** `atendai/evolution-api:latest`
4. **Port:** 8080
5. **Deploy**

### Conectar tu número
1. Abre el dominio que Easypanel te da para Evolution
2. Crea una **Instance** llamada `agentes-ventas`
3. Conecta → escanea el QR con WhatsApp en el móvil del agente

> ⚠️ **Usa una SIM nueva, no tu WhatsApp personal.** Si Meta detecta uso raro, ese número se pierde.

### Conecta Evolution con tu app
En las variables de entorno de tu app en Easypanel:
```
EVOLUTION_API_URL=http://evolution.agente-ventas.svc.cluster.local:8080
EVOLUTION_INSTANCE_NAME=agentes-ventas
EVOLUTION_API_KEY=la_clave_que_te_dio_evolution
```

Reinicia la app en Easypanel.

**✅ Checkpoint:** tu agente recibe mensajes de WhatsApp.

---

## PASO 12 — Configura GHL y haz la prueba final

### Pipeline de ventas
1. En GHL → **Opportunities** → **Pipelines** → **Create Pipeline**
2. Crea estas etapas:
   - Nuevo Lead
   - Recursos Enviados
   - Cualificado
   - Llamada Agendada
   - Descartado
3. Copia el ID de cada etapa (mira la URL al editar) y mételos en las variables de entorno de tu app

### Calendario
1. **Calendars** → crea un calendario para llamadas de ventas
2. Copia el Calendar ID y mételo en `GHL_CALENDAR_ID`

### Prueba completa
Desde otro móvil escribe al número del agente:
1. Manda "hola" → debe responder con saludo natural
2. Cuéntale algo de tu negocio ficticio → debe hacer preguntas
3. Manda un audio → debe entenderlo
4. Cuando proponga llamada, agenda → debe aparecer en GHL Calendar
5. Comprueba en GHL: contacto creado, en la etapa correcta, con notas

Si los 5 puntos funcionan: **lo tienes**.

---

## Comandos del día a día

### En Easypanel
- **Logs:** botón en cada servicio para ver qué hace en vivo
- **Restart:** reinicia un servicio si se atasca
- **Environment:** edita variables sin tocar código

### Por WhatsApp (desde tu número de admin)
- `/reset 34600000000` → reinicia conversación
- `/help` → comandos disponibles

---

## Cuando algo no funciona

**El agente no responde:**
- Mira los logs del servicio `app` en Easypanel
- Si no aparece nada cuando escribes, comprueba que la instancia de Evolution sigue conectada (a veces el QR caduca)

**Errores de base de datos:**
- Comprueba que las variables `POSTGRES_*` apuntan al servicio `db` de tu proyecto en Easypanel
- En Easypanel los servicios se conectan por nombre interno, no por IP

**El agente habla raro:**
- Vuelve al system prompt en `backend/app/agents/sales/prompts.py`
- Pídele a Claude Code: "ajusta el system prompt para que [lo que falle]"

**Errores que no entiendes:**
- Copia el error y pégaselo a Claude Code: "esto está apareciendo en los logs, arréglalo"
- 90% de las veces te lo soluciona en un par de turnos

---

## Resumen de costes

| Servicio | Mes |
|---|---|
| Contabo VPS S | ~7€ |
| Anthropic API | 20-50€ |
| OpenAI Whisper | 1-5€ |
| GoHighLevel | 97€ |
| Claude Code Pro | 20€ |
| Dominio | ~1€ |
| **Total** | **~146-180€/mes** |

---

## Lo que NO hace este agente todavía

- Llamadas de voz
- Publicación en redes sociales
- Prospección en frío por WhatsApp (eso se hace por email frío)
- Facturación

Cada uno de esos es un departamento independiente que se añade después.

---

## Una última cosa

Si te atascas en algún paso, vuelve al checkpoint del paso anterior. La mayoría de errores vienen de saltarse un detalle pequeño.

Y recuerda: **no estás replicando un programa, estás construyendo el tuyo.** Cuanto más entiendas lo que hace cada pieza, más vas a poder adaptarlo a lo que TU negocio necesita.

Cuando lo tengas funcionando vas a flipar.

---

*Construido siguiendo la metodología Sistema de Agentes · Versión 2.0*
