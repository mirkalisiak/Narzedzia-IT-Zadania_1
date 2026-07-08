# 🚀 Django REST API + Docker Compose

Ten projekt przedstawia konfigurację backendu opartego na **Django + Django REST Framework**, uruchamianego w kontenerach Docker z bazą PostgreSQL.

# 📁 Project structure

```text

├── mysite/
│   ├── manage.py
│   ├── mysite/
│   │   ├── __init__.py
│   │   ├── settings.py
│   │   ├── urls.py
│   │   ├── asgi.py
│   │   └── wsgi.py
│   │
│   ├── polls/
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── serializers.py
│   │   ├── urls.py
│   │   └── apps.py
│   │
│   ├── requirements.txt
|   ├── docker-compose.yml
|    ├── .env
│   └── Dockerfile    
└── README.md
```

## 🐳 Uruchomienie projektu

### 1. Build i start kontenerów

```bash
docker compose up --build

docker compose exec web python manage.py makemigrations
docker compose exec web python manage.py migrate
docker compose exec web python manage.py createsuperuser


| Usługa       | URL                                                        |
| ------------ | ---------------------------------------------------------- |
| Django API   | [http://localhost:8000](http://localhost:8000)             |
| Django Admin | [http://localhost:8000/admin](http://localhost:8000/admin) |
| PostgreSQL   | localhost:5432                                             |


DEBUG=1
SECRET_KEY=supersecretkey

DB_NAME=mydb
DB_USER=myuser
DB_PASSWORD=mypassword
DB_HOST=db
DB_PORT=5432


FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["gunicorn", "myproject.wsgi:application", "--bind", "0.0.0.0:8000"]


version: "3.9"

services:
  web:
    build: ./backend
    container_name: django_api
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - ./backend:/app
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      - db

  db:
    image: postgres:15
    container_name: postgres_db
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:


Django>=5.0
djangorestframework
gunicorn
psycopg2-binary

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('DB_NAME'),
        'USER': os.getenv('DB_USER'),
        'PASSWORD': os.getenv('DB_PASSWORD'),
        'HOST': os.getenv('DB_HOST'),
        'PORT': os.getenv('DB_PORT'),
    }
}


INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',

    'rest_framework',
    'api',
]
