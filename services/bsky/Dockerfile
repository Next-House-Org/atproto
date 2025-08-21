FROM node:20.11-alpine as build

# Install build tools
RUN apk add --no-cache python3 make g++ git bash curl jq

RUN corepack enable

WORKDIR /app

COPY ./package.json ./
RUN corepack prepare --activate

# Move files into the image and install
COPY ./*.* ./
# NOTE bsky's transitive dependencies go here: if that changes, this needs to be updated.
COPY ./tsconfig ./tsconfig
COPY ./packages/bsky ./packages/bsky
COPY ./packages/api ./packages/api
COPY ./packages/common ./packages/common
COPY ./packages/common-web ./packages/common-web
COPY ./packages/crypto ./packages/crypto
COPY ./packages/did ./packages/did
COPY ./packages/identity ./packages/identity
COPY ./packages/sync ./packages/sync
COPY ./packages/syntax ./packages/syntax
COPY ./packages/lexicon ./packages/lexicon
COPY ./packages/repo ./packages/repo
COPY ./packages/xrpc ./packages/xrpc
COPY ./packages/xrpc-server ./packages/xrpc-server
COPY ./packages/internal/fetch-node ./packages/internal/fetch-node
COPY ./packages/internal/fetch ./packages/internal/fetch
COPY ./packages/internal/pipe ./packages/internal/pipe
COPY ./packages/internal/xrpc-utils ./packages/internal/xrpc-utils
COPY ./services/bsky ./services/bsky

# install all deps
RUN pnpm install --frozen-lockfile > /dev/null
# build all packages with external node_modules
RUN pnpm build > /dev/null
# clean up
RUN rm -rf node_modules
# install only prod deps, hoisted to root node_modules dir
RUN pnpm install --prod --shamefully-hoist --frozen-lockfile --prefer-offline > /dev/null

WORKDIR services/bsky

# Uses assets from build stage to reduce build size
FROM node:20.11-alpine

# Install runtime tools and utilities
RUN apk add --no-cache \
    dumb-init \
    curl \
    openssl \
    coreutils \
    vim bash git jq \
    sqlite \
    postgresql-client \
    runit

# Create a non-root user and set permissions
RUN addgroup -S appuser && \
    adduser -S -G appuser -h /app appuser && \
    chown -R appuser:appuser /app

USER appuser

WORKDIR /app/services/bsky
COPY --from=build /app /app

EXPOSE 3000
ENV PORT=3000
ENV NODE_ENV=production

# https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md#non-root-user
CMD ["dumb-init", "--", "node", "--heapsnapshot-signal=SIGUSR2", "--enable-source-maps", "api.js"]

LABEL org.opencontainers.image.source=https://github.com/Next-House-Org/atproto
LABEL org.opencontainers.image.description="bluesky-appview"
LABEL org.opencontainers.image.licenses=MIT
