ARG NODE_ENV=production

FROM node:18-bookworm-slim@sha256:c569d19289293797aefad0473a046d882f1b9eb0f444e9a0018648d5fd03c812 as build-stage

ARG NODE_ENV
ENV NODE_ENV=${NODE_ENV}

WORKDIR /speckle-server

COPY .yarnrc.yml .
COPY .yarn ./.yarn
COPY package.json yarn.lock ./

COPY packages/frontend-2/type-augmentations/stubs ./packages/frontend-2/type-augmentations/stubs/
COPY packages/webhook-service/package.json ./packages/webhook-service/
COPY packages/shared/package.json ./packages/shared/

RUN yarn workspaces focus --all

COPY packages/shared ./packages/shared/
COPY packages/webhook-service/src ./packages/webhook-service/

RUN yarn workspaces foreach run build

ARG WAIT_VERSION=2.8.0
ENV WAIT_VERSION=${WAIT_VERSION}
ADD https://github.com/ufoscout/docker-compose-wait/releases/download/${WAIT_VERSION}/wait ./wait
RUN chmod +x ./wait

ARG TINI_VERSION=v0.19.0
ENV TINI_VERSION=${TINI_VERSION}
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini ./tini
RUN chmod +x ./tini

FROM node:18-bookworm-slim@sha256:c569d19289293797aefad0473a046d882f1b9eb0f444e9a0018648d5fd03c812 as dependency-stage
# yarn install
ARG NODE_ENV
ENV NODE_ENV=${NODE_ENV}

WORKDIR /speckle-server

COPY .yarnrc.yml .
COPY .yarn ./.yarn
COPY package.json yarn.lock ./

COPY packages/frontend-2/type-augmentations/stubs ./packages/frontend-2/type-augmentations/stubs/
COPY packages/webhook-service/package.json ./packages/webhook-service/
COPY packages/shared/package.json ./packages/shared/

WORKDIR /speckle-server/packages/webhook-service
RUN yarn workspaces focus --production

FROM gcr.io/distroless/nodejs18-debian12:nonroot@sha256:d0a14121fb93c8b71c6f2565ef9dc02afb223d5f8de7cb50fd37c027aad237d9 as production-stage
ARG NODE_ENV
ENV NODE_ENV=${NODE_ENV}

WORKDIR /speckle-server/packages/webhook-service/
COPY packages/webhook-service/src .
COPY --from=build-stage /speckle-server/wait /usr/bin/wait
COPY --from=build-stage /speckle-server/tini /usr/bin/tini
COPY --from=build-stage /speckle-server/packages/shared ./packages/shared
COPY --from=build-stage /speckle-server/packages/webhook-service ./packages/webhook-service
COPY --from=dependency-stage /speckle-server/node_modules ./node_modules

ENTRYPOINT [ "tini", "--", "/nodejs/bin/node", "main.js"]
