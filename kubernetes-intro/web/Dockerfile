FROM python:3.7.0-alpine
ADD app/ /app
WORKDIR /app
USER 1001:1001
EXPOSE 8000
ENTRYPOINT ["python3", "-m", "http.server", "8000", "--directory", "/app"]
