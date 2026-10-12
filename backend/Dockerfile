# Backend Dockerfile for PrivacyLens API
# Multi-stage build for smaller production image

# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

# Copy all necessary files for backend build
# Backend needs access to both backend/ and src/ directories
COPY backend/package*.json ./backend/
COPY backend/tsconfig.json ./backend/
COPY backend/src/ ./backend/src/
COPY src/privacy-engine/ ./src/privacy-engine/

# Install dependencies in backend directory
WORKDIR /app/backend
RUN npm install --production=false

# Build the application from backend directory
RUN npm run build

# Production stage
FROM node:20-alpine AS production

WORKDIR /app

# Copy built files from builder
COPY --from=builder /app/backend/node_modules ./backend/node_modules
COPY --from=builder /app/backend/dist ./backend/dist
COPY --from=builder /app/backend/package*.json ./backend/

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

# Expose port
EXPOSE 4000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:4000/health || exit 1

# Start the application
WORKDIR /app/backend
CMD ["node", "dist/backend/src/index.js"]
