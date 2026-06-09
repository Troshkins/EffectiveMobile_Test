FROM python:3.12-slim

WORKDIR /app

RUN useradd --create-home --shell /usr/sbin/nologin appuser

COPY app.py .

USER appuser

EXPOSE 8080

CMD ["python", "app.py"]

