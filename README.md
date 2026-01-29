# airflow-gcp-lab
repocitorio para laboratorio AirFlow para bootcamp DEPIA 29

## Nombre: Marcos Ernesto Campos Remirez


## Descripcion del proyecto

La solución airflow-gcp-lab es un pipeline de datos automatizado construido con Apache Airflow, diseñado para procesar datos de ventas en tiempo real. Utiliza Google Cloud Platform (GCP) para integrar servicios como Pub/Sub (para recepción de mensajes), Cloud Storage (para almacenamiento intermedio) y BigQuery (para análisis y almacenamiento final). El sistema incluye:

Un DAG (Directed Acyclic Graph) que extrae mensajes de Pub/Sub, los procesa en formato JSON, los sube a GCS y los carga en BigQuery, creando tablas y datasets según sea necesario.
Una API de ventas simulada en un contenedor separado para generar datos de prueba.
Un entorno Docker Compose que orquesta Airflow con servicios como PostgreSQL y Redis para la ejecución distribuida.
Esta solución facilita el procesamiento ETL (Extract, Transform, Load) de datos de ventas, asegurando escalabilidad y fiabilidad en la nube.

## Como Ejecutar 

### Inicializar Airflow: Antes de levantar todo, debemos preparar la base de datos y crear el usuario administrador.
docker-compose up airflow-init

Espera hasta que veas el mensaje airflow-init_1 exited with code 0.

 ### Levantar los servicios: Ahora sí, levantamos todo el clúster en segundo plano:
 
docker-compose up -d
### Verifica:

Airflow: http://localhost:8080 (User/Pass: airflow / airflow).

API Sales: http://localhost:8081/docs.

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
