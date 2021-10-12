FROM node:12.18.2 as build

WORKDIR /app

COPY package.json /app/package.json
COPY package-lock.json /app/package-lock.json

RUN npm install

COPY . .

RUN npm run build

FROM nginx

COPY --from=build /assets/bundles /usr/share/nginx/html