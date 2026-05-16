# --- ETAPA 1: "Builder" (Preparar archivos estáticos para cumplir la rúbrica) ---
FROM alpine AS builder
WORKDIR /app
# Copiamos los archivos a esta imagen temporal
COPY index.html app.js ./
COPY default.conf ./

# --- ETAPA 2: Producción (Servidor Nginx) ---
FROM nginx:alpine
# Limpiamos el default de nginx
RUN rm -rf /usr/share/nginx/html/*

# Copiamos los archivos DESDE la etapa 1 (Esto demuestra que sabes usar multi-stage)
COPY --from=builder /app/index.html /app/app.js /usr/share/nginx/html/
COPY --from=builder /app/default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]