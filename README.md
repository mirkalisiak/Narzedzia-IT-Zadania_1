# Narzedzia-IT-Zadania_1
1. Otwieramy terminal i wpisujemy poniższą komendę, która pobierze obraz i zmapuje odpowiednie porty. Po uruchomieniu panel webowy będzie dostępny w przeglądarce pod adresem http://localhost:8080.

docker run -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts

2. Tworzymy nowe zadanie wybierając opcję Freestyle project. W konfiguracji zjeżdżamy do sekcji budowania, gdzie dodajemy krok Execute shell. Poniższy skrypt po prostu wypisze zadany tekst w logach z budowania, co pozwala zweryfikować, czy Jenkins poprawnie wykonuje nasze polecenia.

echo "Hello Jenkins"

3. Aby uelastycznić zadania, możemy przekazywać do nich dane wejściowe. W konfiguracji zaznaczamy opcję parametryzacji projektu i dodajemy parametr tekstowy, nadając mu nazwę `NAME`. W kroku Execute shell odwołujemy się do tej zmiennej za pomocą znaku dolara, dzięki czemu powitanie jest dynamiczne i zależy od tego, co wpisze użytkownik przy uruchamianiu.

echo "Witaj $NAME"

4. W konfiguracji zadania zaznaczamy Git i wklejamy adres URL naszego repozytorium. Jenkins automatycznie pobierze kod. Następnie w kroku Execute shell używamy standardowych komend systemowych do wyświetlenia pobranej zawartości.

ls -la

5. Poniższy skrypt definiuje środowisko wykonawcze oraz pojedynczy etap, który wykonuje proste polecenie wypisania tekstu.

pipeline {
    agent any
    stages {
        stage('Hello') {
            steps {
                echo 'Hello from Pipeline'
            }
        }
    }
}

6. W tym zadaniu rozbudowujemy strukturę o dwa nowe bloki: etap Build oraz etap Test. Rozdzielenie tych akcji sprawia, że w przypadku błędu od razu widzimy wizualnie, w której fazie projekt napotkał problem.

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
            }
        }
    }
}

7. W pierwszym etapie tworzymy przykładowy plik tekstowy. Następnie, wykorzystując sekcję `post`, instruujemy Jenkinsa, by za pomocą polecenia zapisał ten wynik po zakończeniu zadania i umożliwił jego pobranie.

pipeline {
    agent any
    stages {
        stage('Generowanie pliku') {
            steps {
                sh 'echo "To jest wynik" > output.txt'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'output.txt', followSymlinks: false
        }
    }
}

8.Definicja pobiera projekt Django ze wskazanego adresu URL , wylistowuje pliki dla weryfikacji , a następnie buduje z niego obraz Dockerowy. Obraz jest pakowany do pliku `.tar`, co pozwala na jego finalną archiwizację.

pipeline {
    agent any
    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/Matiorus/Django-build.git'
            }
        }
        stage('List Files') {
            steps {
                sh 'ls -la'
            }
        }
        stage('Build Django in Docker') {
            steps {
                // Budowanie obrazu na podstawie Dockerfile z repozytorium
                sh 'docker build -t django-app .'
                
                // Zapis obrazu do pliku tar w celu archiwizacji
                sh 'docker save django-app > django-app.tar'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'django-app.tar', followSymlinks: false




---------------------------------------------------------------------------------------------------------------------------------------

1. git clone https://github.com/Matiorus/Django-build.git
cd Django-build

2.pip install -r requirements.txt
pip install psycopg2-binary
python manage.py migrate
python manage.py runserver

3.
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]


docker build -t django-sqlite .
docker run -p 8000:8000 django-sqlite


4.Zgodnie z poleceniem definiujemy dwie odrębne sieci (docker network): jedna dla Django i postgre, druga dla django i pgadmin. Dodatkowo zablokujemy eksponowanie bazy danych, po prostu pomijając w definicji bazy sekcję odpowiedzialną za mapowanie portów na maszynę hosta.

version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: mydatabase
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
    networks:
      - django_postgres_net
    # [cite_start]Brak sekcji "ports" celowo blokuje eksponowanie bazy na zewnątrz [cite: 85]

  web:
    build: . # [cite_start]Zbuduje obraz na podstawie wymaganego pliku Dockerfile [cite: 82]
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    environment:
      - DB_HOST=db
      - DB_NAME=mydatabase
      - DB_USER=myuser
      - DB_PASS=mypassword
    depends_on:
      - db
    networks:
      # Kontener z Django przypisany jest do obu wymaganych sieci
      - django_postgres_net
      - django_pgadmin_net

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: root
    ports:
      - "5050:80"
    networks:
      - django_pgadmin_net
      # Uwaga z zakresu administracji: pgadmin musi być również w sieci bazy, aby mógł fizycznie nawiązać z nią połączenie.
      - django_postgres_net 

networks:
  [cite_start]django_postgres_net: # Sieć przeznaczona dla komunikacji Django i PostgreSQL [cite: 84]
  [cite_start]django_pgadmin_net:  # Sieć przeznaczona dla komunikacji Django i pgAdmin [cite: 84]
        }
    }
}
