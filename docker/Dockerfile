FROM alpine:3.18.4 as base

WORKDIR /app
RUN apk add --no-cache tzdata eudev tini nodejs

# Dependencies and build
FROM base as dependencies_and_build

COPY package*.json tsconfig.json index.js ./
COPY lib ./lib

RUN apk add --no-cache --virtual .buildtools make gcc g++ python3 linux-headers git npm && \
    npm ci && \
    npm run build && \
    npm ci --omit=dev,optional --audit=false && \
    apk del .buildtools

# Release
FROM base as release

COPY --from=dependencies_and_build /app/node_modules ./node_modules
COPY --from=dependencies_and_build /app/dist ./dist
COPY package.json LICENSE index.js data/configuration.yaml data/configuration.example.yaml ./

COPY docker/docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

RUN mkdir /app/data

ARG COMMIT
RUN echo "$COMMIT" > dist/.hash

ENV NODE_ENV production

ENTRYPOINT ["docker-entrypoint.sh"]
CMD [ "/sbin/tini", "--", "node", "index.js"]
