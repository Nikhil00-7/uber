FROM node:20-alpine

WORKDIR /app

# Install build tools for native modules
RUN apk add --no-cache python3 make g++ bash

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3002
CMD ["node", "server.js"]
