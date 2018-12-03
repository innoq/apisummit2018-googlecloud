# Demo Datastores

## gcloud konfigurieren
`gcloud auth login`
`gcloud config set project <projectname>`


## Cloud SQL
* Kommandzeilentools: `gcloud sql`
* Postgres Kommandozeilenclient: `psql`
* Vorhandene Cloud SQL Instanzen auflisten: `gcloud sql instances list`

### Instanz anlegen
* Name: todo-pg-instance
* Typ: Postgres
* Wird keine Region angegeben, wird als Default eine US Region gewählt
  * Liste aller Regionen: https://cloud.google.com/about/locations/?region=europe#region
* Kleinstmögliches Tier
  * Liste aller Tiers: https://cloud.google.com/sql/pricing
* `gcloud sql instances create todo-pg-instance --tier=db-f1-micro --region=europe-west3 --database-version=POSTGRES_9_6` alternativ:
`gcloud sql instances create myinstance --cpu=2 --memory=7680MiB --database-version=POSTGRES_9_6`

DAUERT EWIG, DESHALB HIER WECHSEL ZU CLOUD STORAGE


## Cloud Storage
* Wir verwenden `gsutil` ein Kommandozeilentools, dass zusammen mit dem Google Cloud SDK installiert wird
* Gedacht für die Administration von Cloud Storage
* Funktioniert auch mit Amazon S3

### Setup prüfen
* Ausprobieren: Welche Buckets sind schon da? `gsutil ls`
* Über Oberfläche verifizieren: https://console.cloud.google.com

### Bucket anlegen
* Name: hallo-apisummit2018
* Storage class: multi-regional
* Location: europe
* `gsutil mb -l eu gs://hallo-apisummit2018`
* Benamung: `gs://<bucketname>`
* Wenn als Location eine multi-region angegeben wird, wird automatisch die Storage Klasse multi-regional verwendet
* Mit `gsutil ls` und über Oberfläche verifizieren

### Initiale Daten hochladen
* Daten sollen im Verzeichnis `initial_data` liegen
* Hochzuladende Datei: `todos.sql`
* `gsutil cp todos.sql gs://hallo-apisummit2018/initial_data/`
* Dateien im Bucket auflisten `gsutil ls gs://hallo-apisummit2018/**`

ZURÜCK ZU CLOUD SQL
## Cloud SQL
Instanz begutachten
* Auflisten: `gcloud sql instances list`
* Details: `gcloud sql instances describe todo-pg-instance`

### Postgres Benutzer anlegen
`gcloud sql users set-password postgres --instance=todo-pg-instance --password=b5bx7DZfBK5opY`
`gcloud sql users list --instance=todo-pg-instance`

### Datenbank anlegen
* psql Cheatsheet: https://gist.github.com/Kartones/dd3ff5ec5ea238d4c546
* Zu Instanz verbinden: `gcloud sql connect todo-pg-instance --user=postgres`
* Datenbank anlegen: `CREATE DATABASE tododb;`
* Datenbanken auflisten: `\l`
* Zu Datenbank verbinden: `\c tododb`
* Tabellen auflisten: `\dt`
* Tabelle anlegen: `CREATE TABLE todos (id CHAR(36) PRIMARY KEY, todo VARCHAR(255) NULL, created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP);`
* Tabellen auflisten: `\dt`
* `SELECT * FROM todos;``
* postgres Benutzer und Daten über die Oberfläche zeigen
* Ausgeführte Operationen anzeigen: `gcloud sql operations list --instance=todo-pg-instance;`

### Testdaten einspielen
* Daten aus `todos.sql`
* Berechtigungen setzen
  * `gcloud sql instances describe todo-pg-instance`
  * Service Account Email kopieren: qz6kfmulszandcyi34owkshaki@speckle-umbrella-pg-10.iam.gserviceaccount.com
  * `gsutil acl ch -u qz6kfmulszandcyi34owkshaki@speckle-umbrella-pg-10.iam.gserviceaccount.com:W gs://hallo-apisummit2018`
  * Service Account als Reader zum Bucket hinzufügen: `gsutil acl ch -u qz6kfmulszandcyi34owkshaki@speckle-umbrella-pg-10.iam.gserviceaccount.com:R gs://hallo-apisummit2018-2/initial_data/todos.sql`
* `gcloud sql import sql todo-pg-instance gs://hallo-apisummit2018/initial_data/todos.sql --database=tododb --user=postgres`
* Daten anschauen
  * `gcloud sql connect todo-pg-instance --user=postgres`
  * `\c tododb`
  * `select count(*) from todos;`

## Daten aus Datenbank exportieren
* Exportieren als CSV
* `gcloud sql export csv todo-pg-instance gs://hallo-apisummit2018/export/2018-12-03.csv --query="select * from todos" --database=tododb`
* bei Permission Fehler nochmal: `gsutil acl ch -u qz6kfmulszandcyi34owkshaki@speckle-umbrella-pg-10.iam.gserviceaccount.com:W gs://hallo-apisummit2018` ausführen
