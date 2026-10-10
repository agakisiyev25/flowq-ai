# Build the frontend. An empty VITE_API_BASE makes the browser call the API on the page's own origin.
FROM node:22-slim AS frontend
WORKDIR /app/frontend
COPY frontend/package.json frontend/package-lock.json ./
RUN npm ci
COPY frontend/ ./
RUN VITE_API_BASE= VITE_MOCK=0 npm run build

FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PORT=8000 \
    DATABASE_PATH=/data/flowq.sqlite3 \
    UPLOAD_DIR=/data/media \
    FRONTEND_DIST=frontend/dist \
    ALLOWED_ORIGINS=http://localhost:5173

WORKDIR /app
COPY backend/requirements.txt /app/backend/requirements.txt
RUN pip install --no-cache-dir -r backend/requirements.txt \
    && useradd --uid 10001 --create-home flowq \
    && mkdir -p /data/media \
    && chown -R flowq:flowq /data

COPY backend/*.py /app/backend/
COPY backend/tools/*.py /app/backend/tools/
COPY industry_packs/ /app/industry_packs/
COPY --from=frontend /app/frontend/dist/ /app/frontend/dist/

USER flowq
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=5s --start-period=20s --retries=3 \
    CMD python -c "import os,urllib.request; urllib.request.urlopen('http://127.0.0.1:'+os.getenv('PORT','8000')+'/api/health',timeout=4)"
CMD ["python", "-m", "backend"]
