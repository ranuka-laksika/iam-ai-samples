FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# Install python-dotenv (only dependency for client)
RUN pip install --upgrade pip
RUN pip install python-dotenv

COPY . .

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser -u 10014 appuser

# Set ownership of app directory
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER 10014

EXPOSE 3000

CMD ["python", "serve.py"]
