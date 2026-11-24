# ---- build: install wheels into a virtualenv ----
FROM python:3.12-slim AS build
ENV PIP_NO_CACHE_DIR=1 PIP_DISABLE_PIP_VERSION_CHECK=1
RUN python -m venv /venv
ENV PATH=/venv/bin:$PATH
COPY requirements.txt .
RUN pip install -r requirements.txt

# ---- run ----
FROM python:3.12-slim
ENV PATH=/venv/bin:$PATH \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    DJANGO_SETTINGS_MODULE=media_service.settings
RUN useradd --create-home --uid 1000 app
WORKDIR /app
COPY --from=build /venv /venv
COPY --chown=app:app . .
USER app
EXPOSE 8000
HEALTHCHECK --interval=10s --timeout=3s --start-period=20s --retries=5 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/health')" || exit 1
# API by default. The worker reuses this image with: command: python manage.py consume_events
CMD ["sh", "-c", "python manage.py migrate --noinput && python manage.py init_storage && exec gunicorn media_service.wsgi:application --bind 0.0.0.0:8000 --workers 3 --timeout 600"]
