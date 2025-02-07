# ============================
# 1️⃣ Base Stage - Install Dependencies
# ============================
FROM node:18-alpine AS deps

WORKDIR /app

# Install system dependencies
RUN apk add --no-cache libc6-compat

# Install dependencies based on the package.json and lock file
COPY package.json package-lock.json* yarn.lock* ./
RUN if [ -f yarn.lock ]; then yarn install --frozen-lockfile; elif [ -f package-lock.json ]; then npm ci; else npm install; fi


# ============================
# 2️⃣ Build Stage - Create Optimized Production Build
# ============================
FROM node:18-alpine AS builder

WORKDIR /app

# Copy dependencies from previous stage
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build the Next.js app (output goes to .next)
RUN npm run build

# Remove development dependencies
RUN npm prune --production


# ============================
# 3️⃣ Production Stage - Serve Next.js App Efficiently
# ============================
FROM node:18-alpine AS runtime

WORKDIR /app

# Install smallest possible dependencies for runtime
RUN apk add --no-cache tini

# Copy only the built app and dependencies
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

# Expose Next.js port
EXPOSE 3000

# Use tini to prevent zombie processes
ENTRYPOINT ["/sbin/tini", "--"]

# Default command for production
CMD ["npm", "run", "start"]
