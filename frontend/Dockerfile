# syntax=docker/dockerfile:1
FROM node:20-alpine AS builder

WORKDIR /app

# Copiar dependencias primero (caching de capas).
# Cache mount de BuildKit sobre /root/.npm: los paquetes descargados persisten
# entre builds → npm ci reutiliza la caché y no re-descarga.
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm npm ci --prefer-offline

# Build arg para la URL de la API — en Docker usa ruta relativa (nginx proxy)
ARG  VITE_API_BASE_URL=/api/v1
ENV  VITE_API_BASE_URL=$VITE_API_BASE_URL

# Copiar código y compilar
COPY . .
RUN npm run build

# ── Etapa 2: Servir con nginx ──────────────────────────────────────────────
FROM nginx:1.25-alpine

# Copiar build de Vite al directorio de nginx
COPY --from=builder /app/dist /usr/share/nginx/html

# Copiar configuración de nginx como template — el entrypoint oficial de la imagen
# (docker-entrypoint.d/20-envsubst-on-templates.sh) la resuelve a conf.d/default.conf
# al arrancar el contenedor, sustituyendo STORAGE_MAX_FILE_SIZE_MB por su valor real
# (docker-compose.yml se lo pasa). NGINX_ENVSUBST_FILTER limita la sustitución a esa
# única variable para no pisar las variables propias de nginx ($host, $remote_addr, etc.)
# que también aparecen en el archivo.
COPY nginx.conf.template /etc/nginx/templates/default.conf.template
ENV STORAGE_MAX_FILE_SIZE_MB=1000
ENV NGINX_ENVSUBST_FILTER='^STORAGE_MAX_FILE_SIZE_MB$'

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]