FROM python:3.11-slim

# Set environment variables
ENV PYTHONUNBUFFERED=1 \
  PYTHONDONTWRITEBYTECODE=1 \
  ENV_MODE="production" \
  PYTHONPATH=/app

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
  build-essential \
  curl \
  ca-certificates \
  wget \
  && rm -rf /var/lib/apt/lists/*

# Install Supabase CLI via official Linux package
RUN SUPABASE_VERSION=$(curl -s https://api.github.com/repos/supabase/cli/releases/latest | grep '"tag_name":' | sed -E 's/.*"v([^"]+)".*/\1/') \
  && wget "https://github.com/supabase/cli/releases/latest/download/supabase_${SUPABASE_VERSION}_linux_amd64.deb" \
  && dpkg -i "supabase_${SUPABASE_VERSION}_linux_amd64.deb" \
  && rm "supabase_${SUPABASE_VERSION}_linux_amd64.deb"

# Create non-root user and set up directories
RUN useradd -m -u 1000 appuser && \
  mkdir -p /app/logs && \
  chown -R appuser:appuser /app

# Install Python dependencies
COPY --chown=appuser:appuser requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt gunicorn

# Switch to non-root user
USER appuser

# Copy application code
COPY --chown=appuser:appuser . .

# Run Supabase migrations after copying the configuration files
RUN supabase db push

# Expose the port the app runs on
EXPOSE 8000

# Calculate optimal worker count based on 16 vCPUs
# Using (2*CPU)+1 formula for CPU-bound applications
ENV WORKERS=33
ENV THREADS=2
ENV WORKER_CONNECTIONS=2000

# Gunicorn configuration
CMD ["sh", "-c", "gunicorn api:app \
  --workers $WORKERS \
  --worker-class uvicorn.workers.UvicornWorker \
  --bind 0.0.0.0:8000 \
  --timeout 1800 \
  --graceful-timeout 600 \
  --keep-alive 1800 \
  --max-requests 0 \
  --max-requests-jitter 0 \
  --forwarded-allow-ips '*' \
  --worker-connections $WORKER_CONNECTIONS \
  --worker-tmp-dir /dev/shm \
  --preload \
  --log-level info \
  --access-logfile - \
  --error-logfile - \
  --capture-output \
  --enable-stdio-inheritance \
  --threads $THREADS"]
