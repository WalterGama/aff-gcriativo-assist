FROM node:20-slim

WORKDIR /app

# Copy package files first for better layer caching
COPY package*.json ./

# Install build dependencies for node-gyp
RUN apt-get update && apt-get install -y --no-install-recommends \
  python3 \
  make \
  g++ \
  build-essential \
  pkg-config \
  libcairo2-dev \
  libpango1.0-dev \
  libjpeg-dev \
  libgif-dev \
  librsvg2-dev \
  && rm -rf /var/lib/apt/lists/*

RUN npm install

# Copy the frontend code
COPY . .

RUN npm run build

EXPOSE 3000
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

# Default command is dev, but can be overridden in docker-compose
CMD ["npm", "start"]