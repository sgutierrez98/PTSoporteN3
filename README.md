# Prueba Técnica: Ingeniero de Soporte N3 - Diagnóstico de Microservicios

## Escenario
Se ha reportado una caída crítica en el portal de clientes. El equipo de Nivel 2 escaló el caso indicando que, tras el último despliegue, el servicio no responde correctamente. Tu misión es estabilizar el entorno, identificar las fallas y asegurar la comunicación entre los componentes.

## Arquitectura del Entorno
El sistema consta de tres capas corriendo en contenedores Docker:
1. **nginx-proxy**: Balanceador de carga y punto de entrada (Puerto 8080).
2. **api-service**: Lógica de negocio (Node.js).
3. **database**: Motor de base de datos (PostgreSQL).

---

## Solución
1. Versión inválida en docker-compose.yml
Estaba declarado version: '1', que Docker Compose no reconoce. Lo cambié a '3.8'.
2. El host de la base de datos apuntaba a localhost
En Docker los contenedores no se comunican por localhost, sino por el nombre del servicio. La API nunca encontraba la base de datos. Lo corregí a DB_HOST: 'database'.
3. Límite de memoria de 5 MB
Node.js necesita mínimo 30–50 MB para arrancar. Con 5 MB el contenedor moría al instante. Eliminé esa restricción.
4. Nginx apuntaba al puerto equivocado
El upstream tenía api-service:8080 pero la API escucha en el 4500. Cada petición devolvía 502 Bad Gateway. Lo corregí en nginx.conf.

Plus — Healthcheck de PostgreSQL
Agregué pg_isready como healthcheck en la base de datos y condition: service_healthy en la API, para que no intente conectarse antes de que Postgres esté realmente listo.