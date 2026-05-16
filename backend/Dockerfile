# --- ETAPA 1: Builder (Instalación de dependencias) ---
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install --production

# --- ETAPA 2: Producción (Imagen final ligera y segura) ---
FROM node:18-alpine
WORKDIR /app

# Habilitar usuario NO root (por seguridad - exigencia de la rúbrica)
USER node

# Copiar dependencias de la etapa builder
COPY --chown=node:node --from=builder /app/node_modules ./node_modules
# Copiar el resto del código (server.js, etc)
COPY --chown=node:node . .

EXPOSE 3001
CMD ["npm", "start"]