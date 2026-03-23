# Docker Skill

## Descripción

Skill para proporcionar comandos y prácticas recomendadas de Docker dentro del agente Copilot.

## Objetivo

Ayudar al usuario a:

- Escribir y depurar Dockerfile
- Usar docker build / run / compose
- Optimizar imágenes y almacenamiento en caché
- Solucionar errores comunes

## Docker Compose: alcance específico

- Crear `docker-compose.yml` para proyectos multi-servicio (app, DB, cache, redes)
- Configurar reinicios, volúmenes y env vars de forma segura (`.env`, `env_file`)
- Optimizar con `build` + `cache_from`, `depends_on`, y `profiles`
- Convertir `docker run` a Compose y viceversa
- Diagnóstico de Compose: `docker-compose config`, `docker compose ps`, `logs` y `events`

## Instrucciones para el asistente

1. Cuando el usuario pida ayuda con Docker, responde con:
   - Pasos claros y simples
   - Comandos reproducibles
   - Ejemplos de `Dockerfile`, `docker-compose.yml`, `docker build` y `docker run`
2. Evita explicar demasiado los conceptos básicos a menos que el usuario lo pida.
3. Ofrece siempre un bloque de comandos listos para copiar.
4. Recuérdale al usuario que las imágenes deben usar principios de privilegios mínimos:
   - `USER` no-root siempre que sea posible.
   - No ejecutar procesos como root dentro del contenedor.
   - Evitar capacidades privilegiadas (`--privileged`) sin necesidad.
   - Uso de `chmod`/`chown` en build para asegurar acceso correcto a archivos.

## Ejemplo de interacción

- Usuario: "Necesito un Dockerfile para una app Node.js que use npm ci y cache en un solo layer"
- Respuesta deseada:
  - Dockerfile sugerido
  - Explicación concisa de caching
  - Comando `docker build -t app:latest .`

- Usuario: "Tengo una app web con Nginx + API Python + Postgres, dame un docker-compose.yml"
- Respuesta deseada:
  - `docker-compose.yml` de 3 servicios
  - Uso de `depends_on`, `volumes` y `restart: unless-stopped`
  - Comandos de ejecución `docker compose up -d` y validación `docker compose ps`

## Formato de salida preferido

- Título corto (p.ej., `docker build`)
- Etiquetas de sintaxis markdown
- Atajos de comandos
- Sugerencias de validación (p.ej., `docker inspect`, `docker logs`)
