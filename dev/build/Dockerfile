# ====================
# --- Build Assets ---
# ====================
FROM node:20-alpine AS assets

RUN apk add yarn g++ make cmake python3 --no-cache

WORKDIR /wiki

# Copy only necessary files for yarn install (to optimize caching)
COPY package.json yarn.lock ./

# Install dependencies first (cached unless package.json changes)
RUN yarn --frozen-lockfile --non-interactive

# Now copy the rest of the files
COPY . .

# Clear old built assets (forces fresh favicon & static files)
RUN rm -rf ./assets

# Rebuild everything (including favicon)
RUN yarn build

# Cleanup dev dependencies
RUN rm -rf /wiki/node_modules
RUN yarn --production --frozen-lockfile --non-interactive
RUN yarn patch-package

# ===============
# --- Release ---
# ===============
FROM node:20-alpine
LABEL maintainer="requarks.io"

RUN apk add bash curl git openssh gnupg sqlite --no-cache && \
    mkdir -p /wiki && \
    mkdir -p /logs && \
    mkdir -p /wiki/data/content && \
    chown -R node:node /wiki /logs

WORKDIR /wiki

COPY --chown=node:node --from=assets /wiki/assets ./assets
COPY --chown=node:node --from=assets /wiki/node_modules ./node_modules
COPY --chown=node:node ./server ./server
COPY --chown=node:node --from=assets /wiki/server/views ./server/views
COPY --chown=node:node ./dev/build/config.yml ./config.yml
COPY --chown=node:node ./package.json ./package.json
COPY --chown=node:node ./LICENSE ./LICENSE

USER node

VOLUME ["/wiki/data/content"]

EXPOSE 3000
EXPOSE 3443

# HEALTHCHECK --interval=30s --timeout=30s --start-period=30s --retries=3 CMD curl -f http://localhost:3000/healthz

CMD ["node", "server"]
