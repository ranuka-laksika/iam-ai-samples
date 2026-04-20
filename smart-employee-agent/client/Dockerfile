FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# Install python-dotenv (only dependency for client)
RUN pip install --upgrade pip
RUN pip install python-dotenv

COPY . .

EXPOSE 3000

CMD ["python", "serve.py"]
