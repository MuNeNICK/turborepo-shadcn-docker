# Base image
FROM node:20-alpine AS base

# Set APP_NAME as build argument with a default value
ARG APP_NAME=docs

# Only run if dependency installation is needed
FROM base AS deps
# Install libc6-compat if needed
RUN apk add --no-cache libc6-compat
WORKDIR /app
# Install pnpm and turbo globally
RUN corepack enable pnpm
RUN npm install -g turbo
# Copy only package manager files
COPY package.json pnpm-lock.yaml ./
# Install dependencies
RUN pnpm install --frozen-lockfile

# Only run if source code rebuild is needed
FROM base AS builder
WORKDIR /app
# Install pnpm and turbo globally
RUN corepack enable pnpm
RUN npm install -g turbo
# Copy application code
COPY . .
# Remove unnecessary parts of the monorepo
RUN turbo prune --scope=$APP_NAME --docker
# Install pruned dependencies
RUN pnpm install --frozen-lockfile
# Build the application
RUN turbo build --filter=$APP_NAME...

# Production image
FROM base AS runner
WORKDIR /app
ENV NODE_ENV production
# Create a non-root user for improved security
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
# Copy necessary files from builder stage
COPY --from=builder /app/apps/$APP_NAME/next.config.mjs ./
COPY --from=builder /app/apps/$APP_NAME/package.json ./
# Use output traces to reduce image size automatically
COPY --from=builder --chown=nextjs:nodejs /app/apps/$APP_NAME/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/apps/$APP_NAME/.next/static ./apps/$APP_NAME/.next/static
# Switch to non-root user
USER nextjs
# Expose port
EXPOSE 3000
ENV PORT 3000
# Start the application
CMD ["node", "apps/$APP_NAME/server.js"]
