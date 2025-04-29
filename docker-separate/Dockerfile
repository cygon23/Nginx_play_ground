FROM node:20-alpine

COPY package.json package-lock.json* /app/
COPY src /app/

WORKDIR /app

RUN npm install

CMD [ "node", "server.js" ]