# Nivel 2 — Spring AI + Groq
## Desarrollo Web Avanzado — Laboratorio LLM con Spring Boot

---

## ¿Por qué Groq con Spring AI?

Groq expone una API **100% compatible con OpenAI ChatCompletions**. Spring AI soporta
esto de forma nativa: solo se cambia la `base-url` y el `api-key` en `application.yml`.
**El código Java es idéntico** al que usarías con OpenAI, Mistral o cualquier otro proveedor.

---

## Estructura del Proyecto

```
nivel2-springai/
├── pom.xml                                         ← Spring AI BOM + spring-ai-starter-model-openai
└── src/main/
    ├── java/com/universidad/chatbot/
    │   ├── Nivel2SpringaiApplication.java           ← Punto de entrada
    │   ├── config/
    │   │   ├── ChatDtos.java                        ← Records: ChatRequest y ChatResponse
    │   │   └── GlobalExceptionHandler.java          ← Manejo de errores amigable
    │   ├── service/
    │   │   └── SpringAiChatService.java             ← ⭐ NÚCLEO: ChatClient de Spring AI
    │   └── controller/
    │       └── SpringAiChatController.java          ← Endpoints REST /api/v2/chat
    └── resources/
        └── application.yml                          ← base-url de Groq + api-key
```

---

## Pasos para Ejecutar

### 1. Obtener API Key de Groq (3 minutos, sin tarjeta)

1. Ir a **https://console.groq.com** → Sign Up con Google o GitHub
2. Menú lateral → **API Keys** → **Create API Key**
3. Copiar la key inmediatamente (formato: `gsk_...`) — Groq la muestra una sola vez

### 2. Configurar la variable de entorno en IntelliJ IDEA

```
Run → Edit Configurations → Nivel2SpringaiApplication
→ Environment Variables → agregar:
   GROQ_API_KEY=gsk_TU_KEY_AQUI
```

> ⚠️ Nunca pongas la API Key directamente en `application.yml` ni en el código.
> Esa línea podría terminar en Git accidentalmente.

### 3. Abrir el proyecto y ejecutar

```
File → Open → seleccionar carpeta nivel2-springai
Run → Nivel2SpringaiApplication
```

---

## Probar los Endpoints

### Endpoint 1 — POST con JSON (principal)

```bash
curl -X POST http://localhost:8080/api/v2/chat \
     -H "Content-Type: application/json" \
     -d '{"pregunta": "¿Qué es Spring Boot en 2 oraciones?", "dominio": "Java"}'
```

**Respuesta esperada:**
```json
{
  "respuesta": "Spring Boot es un framework...",
  "modelo": "llama-3.3-70b-versatile",
  "dominio": "Java"
}
```

### Endpoint 2 — POST con parámetros URL

```bash
curl -X POST "http://localhost:8080/api/v2/chat/rapido?pregunta=Explica%20Maven&dominio=Java"
```

### Endpoint 3 — GET verificación de salud

```bash
curl http://localhost:8080/api/v2/chat/salud
```

---

## Ejercicio Clave: Cambiar de Proveedor LLM

El código Java **no cambia**. Solo se modifican `application.yml` y el `pom.xml`.

### De Groq → Ollama (local, sin internet)

1. Reemplazar en `pom.xml`:
```xml
<!-- Quitar: -->
<artifactId>spring-ai-starter-model-openai</artifactId>
<!-- Poner: -->
<artifactId>spring-ai-starter-model-ollama</artifactId>
```

2. Reemplazar en `application.yml`:
```yaml
spring:
  ai:
    ollama:
      base-url: http://localhost:11434
      chat:
        options:
          model: llama3.2:1b
```

3. `SpringAiChatService.java` → **sin cambios** ✅

### De Groq → OpenAI

Solo cambia la `base-url` y el modelo en `application.yml`:
```yaml
spring:
  ai:
    openai:
      api-key: ${OPENAI_API_KEY}
      # base-url: no se pone (usa la de OpenAI por defecto)
      chat:
        options:
          model: gpt-4o-mini
```

---

## Troubleshooting

| Error | Causa | Solución |
|-------|-------|----------|
| `401 Unauthorized` | API Key inválida o no configurada | Verificar variable de entorno `GROQ_API_KEY` |
| `model not found` | Nombre de modelo incorrecto | Usar `llama-3.3-70b-versatile` exactamente |
| `429 Too Many Requests` | Límite del free tier superado | Esperar unos minutos (~14.400 req/día) |
| Puerto 8080 ocupado | Otro proceso usa el puerto | Cambiar `server.port` en `application.yml` |

