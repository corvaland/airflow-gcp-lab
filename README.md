# airflow-gcp-lab
repocitorio para laboratorio AirFlow para bootcamp DEPIA 29

## Nombre: Marcos Ernesto Campos Remirez


## Descripcion del proyecto

La solución airflow-gcp-lab es un pipeline de datos automatizado construido con Apache Airflow, diseñado para procesar datos de ventas en tiempo real. Utiliza Google Cloud Platform (GCP) para integrar servicios como Pub/Sub (para recepción de mensajes), Cloud Storage (para almacenamiento intermedio) y BigQuery (para análisis y almacenamiento final). El sistema incluye:

Un DAG (Directed Acyclic Graph) que extrae mensajes de Pub/Sub, los procesa en formato JSON, los sube a GCS y los carga en BigQuery, creando tablas y datasets según sea necesario.
Una API de ventas simulada en un contenedor separado para generar datos de prueba.
Un entorno Docker Compose que orquesta Airflow con servicios como PostgreSQL y Redis para la ejecución distribuida.
Esta solución facilita el procesamiento ETL (Extract, Transform, Load) de datos de ventas, asegurando escalabilidad y fiabilidad en la nube.

## Requerimientos

### Crear Proyecto  en GCP

1. Ve a la consola de GCP: https://console.cloud.google.com/
2. Haz clic en el selector de proyectos (arriba a la izquierda).
3. Haz clic en "Nuevo Proyecto".
4. Nombre: datapath-lab-airflow (o similar). Anota el ID del Proyecto (lo necesitarás luego).


### Habilitar APIs 

1. En el buscador de la consola, escribe "APIs & Services" y ve a Biblioteca (Library).
2. Busca y habilita las siguientes APIs una por una:
   o Cloud Pub/Sub API
   o Google Cloud Storage JSON API
   o BigQuery API

### Crear Topic y Suscripción (Pub/Sub)

1. Busca "Pub/Sub" en la consola.
2. Ve a Temas (Topics) -> Crear Tema.
   o ID del tema: sales-topic
   o Deja las demás opciones por defecto y crea.
3. Una vez creado, entra al tema sales-topic (o ve a Suscripciones) y haz clic en Crear
Suscripción.
   o ID de la suscripción: sales-sub
   o Tipo de entrega: Pull (Extracción).
   o Haz clic en Crear.

#### Crear Bucket (Cloud Storage)

1. Busca "Cloud Storage" -> Buckets.
2. Haz clic en Crear.
   o Nombre: datapath-sales-bucket-[TU-ID-DE-PROYECTO] (debe ser único globalmente).
   o Ubicación: us-central1 (o la región de tu preferencia).
   o Haz clic en Crear.

#### Crear Dataset (BigQuery)

1. Busca "BigQuery".
2. En el panel "Explorador" (izquierda), haz clic en los tres puntos al lado de tu proyecto -> Crear conjunto de datos (Create dataset).
   o ID del conjunto de datos: sales_dwh
   o Ubicación de datos: US (multiple regions in United States) (o la misma región que tubucket).
   o Haz clic en Crear conjunto de datos.

#### Crear Cuenta de Servicio (IAM)

Para que Airflow y la API puedan conectarse:
1. Ve a IAM y administración -> Cuentas de servicio.
2. Haz clic en Crear cuenta de servicio.
   o Nombre: airflow-lab-sa
   o Haz clic en Crear y continuar.
3. Roles (Permisos): Agrega los siguientes roles:
   o Propietario (Owner) - Para simplificar el lab (o asigna roles específicos: Pub/Sub Editor, Storage Admin, BigQuery Admin).
   o Haz clic en Continuar y luego Listo.
4. Generar Llave JSON:
   o Haz clic en la cuenta de servicio recién creada (airflow-lab-sa@...).
   o Ve a la pestaña Claves (Keys) -> Agregar clave -> Crear clave nueva.
   o Tipo: JSON.
   o Se descargará un archivo a tu computadora.
   o Renombra este archivo a google_credentials.json.
   o Mueve este archivo a la carpeta del laboratorio junto a dockercompose.yaml.

## Como Ejecutar 

### Preparar el entorno (.env): Necesitamos configurar el User ID para que Airflow tenga permisos correctos y definir el ID del proyecto GCP.

o En Linux / Mac: Ejecuta en tu terminal (reemplaza [TU-ID-DE-PROYECTO] por tu ID
real):
     echo -e "AIRFLOW_UID=$(id -u)\nGCP_PROJECT_ID=[TU-ID-DE-PROYECTO]" > .env
     
o En Windows (PowerShell):
     Set-Content .env "AIRFLOW_UID=50000`nGCP_PROJECT_ID=[TU-ID-DE-PROYECTO]"

o Verificación: Abre el archivo .env creado y confirma que tenga dos líneas: una
con AIRFLOW_UID y otra con GCP_PROJECT_ID=....



### Inicializar Airflow: Antes de levantar todo, debemos preparar la base de datos y crear el usuario administrador.
docker-compose up airflow-init

Espera hasta que veas el mensaje airflow-init_1 exited with code 0.

 ### Levantar los servicios: Ahora sí, levantamos todo el clúster en segundo plano:
 
docker-compose up -d
### Verifica:

Airflow: http://localhost:8080 (User/Pass: airflow / airflow).

API Sales: http://localhost:8081/docs.

## Configuración de Airflow

### Variables: En Airflow UI -> Admin -> Variables, crea dos variables:

   o Key: GCP_PROJECT_ID, Val: [TU-ID-DE-PROYECTO]
   o Key: GCS_BUCKET_NAME, Val: [NOMBRE-DE-TU-BUCKET]
### Connections: El docker-compose ya montó tu JSON. Si la conexión automática falla, creamanualmente:
   o Conn Id: google_cloud_default
   o Type: Google Cloud
   o Project Id: [TU-ID-DE-PROYECTO]
   o Keyfile JSON: Pega el contenido de tu google_credentials.json.
### Activar DAG

En la página principal (DAGs), busca sales_pipeline_gcp y enciende el interruptor (ON).

## Resolución de Problemas Comunes

• Error 500 en la API: Si al enviar una orden recibes un error 500, verifica que la
variable GCP_PROJECT_ID en el archivo .env sea correcta y reinicia el contenedor (dockercompose
restart sales-api).
• "No messages found": Si el DAG termina rapido sin procesar, asegúrate de haber enviado
órdenes a la API antes de ejecutar el DAG.

## Ejecución y Pruebas

1. Generar Ventas: Ve a http://localhost:8081/docs, abre el endpoint POST /order -> "Try it out". Envía varios JSONs de prueba (cambia product_id, quantity, etc.).
2. Ejecutar DAG: En Airflow, haz click en el botón "Play" del DAG sales_pipeline_gcp.
3. Verificar: Revisa en tu consola de GCP (Cloud Storage y BigQuery) que los datos hayan llegado.

#### PUB/SUB GCP

<img width="1510" height="634" alt="pub_sub" src="https://github.com/user-attachments/assets/50eebe8c-e739-4306-927e-bb78125905a6" />

#### BigQuery GCP

<img width="1443" height="835" alt="big_query_select" src="https://github.com/user-attachments/assets/6b3a5f8f-7629-4c1a-8472-05f5de020409" />

<img width="1521" height="698" alt="big_query_schema" src="https://github.com/user-attachments/assets/443476c6-6938-4881-9e69-fa69529563c2" />

#### API POST

<img width="1413" height="773" alt="API_OK_POST" src="https://github.com/user-attachments/assets/644be2d4-1020-41fc-88ef-ec6479bb350b" />

#### AIR-FLOW

<img width="1831" height="635" alt="air_flow_dag_green" src="https://github.com/user-attachments/assets/70eaa6f0-b0a4-4b43-b5af-749972f6cb5a" />

<img width="1673" height="796" alt="log_process_and_upload_gcs" src="https://github.com/user-attachments/assets/39fdc64d-ed49-490e-b44b-63d52b6ccbc8" />
