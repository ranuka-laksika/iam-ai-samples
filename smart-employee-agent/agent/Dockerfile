FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    git \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --upgrade pip
RUN pip install 'uvicorn[standard]'

# Install requirements
RUN pip install -r requirements.txt

COPY . .

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser -u 10014 appuser

# Set ownership of app directory
RUN chown -R appuser:appuser /app

# Switch to non-root user
USER 10014

EXPOSE 5001

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "5001"]
