FROM node:22-alpine AS build

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .

FROM node:22-alpine AS dev

WORKDIR /app

COPY --from=build /app /app

USER node

CMD ["npm", "run", "dev"]
