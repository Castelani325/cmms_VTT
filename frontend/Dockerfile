FROM node:21.6.1 AS build

WORKDIR /usr/src/app
COPY package*.json ./
RUN npm config set fetch-retry-maxtimeout 6000000
RUN npm config set fetch-retry-mintimeout 1000000
RUN NODE_OPTIONS="--max_old_space_size=4096" npm install --legacy-peer-deps
COPY . .
ENV PUBLIC_URL=/Atlas
RUN npm run build

FROM nginx:1.27.0-alpine
# Serve os arquivos sob /Atlas para o path correto funcionar no Nginx
COPY --from=build /usr/src/app/build /usr/share/nginx/html/Atlas
COPY --from=build /usr/src/app/.env.example /usr/share/nginx/html/Atlas/.env
COPY --from=build /usr/src/app/nginx-custom.conf /etc/nginx/conf.d/default.conf
RUN chmod -R 755 /usr/share/nginx/html/Atlas

RUN apk add --update nodejs
RUN apk add --update npm

RUN npm i -g runtime-env-cra

WORKDIR /usr/share/nginx/html/Atlas
EXPOSE 3000

CMD ["/bin/sh", "-c", "runtime-env-cra && nginx -g \"daemon off;\""]

