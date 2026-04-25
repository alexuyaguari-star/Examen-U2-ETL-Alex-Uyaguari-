# Examen-U2-ETL-Alex-Uyaguari-Proyecto Integrador: Pipeline de Ingeniería de Datos End-to-End
Introducción
El presente proyecto tiene como objetivo la construcción de un sistema de ingeniería de datos completo bajo la arquitectura Lakehouse. La solución integra procesos de ingesta, validación de calidad, transformación distribuida, modelado dimensional y gobierno de datos, asegurando la privacidad de la información y la trazabilidad del ciclo de vida del dato.
Arquitectura del sistema
La solución se basa en el patrón Write-Audit-Publish (WAP), el cual garantiza la integridad del repositorio de datos mediante una zona de validación intermedia. El flujo se divide en capas de persistencia (Bronze, Silver y Gold), gestionadas por un único orquestador central que asegura la coherencia operativa de todos los componentes.
Cumplimiento de requisitos técnicos
1. Ingesta de datos
La fase de ingesta se desarrolla mediante un script de Python orquestado en Airflow. Se utiliza la librería requests para el consumo de una API REST externa, implementando una lógica de extracción resiliente con reintentos exponenciales para el manejo de límites de tasa (rate limits). Los datos crudos se persisten en una tabla de auditoría inicial, manteniendo su estado original para procesos de re-procesamiento.
2. Calidad de datos y Quality Gate bloqueante
Se ha implementado un proceso de validación utilizando el framework Great Expectations (GX). Este componente actúa como un Quality Gate crítico y bloqueante dentro del pipeline. Mediante el uso de excepciones controladas de Airflow, el sistema garantiza que, si los datos no cumplen con las expectativas de integridad y veracidad predefinidas, el flujo se detiene inmediatamente, evitando la propagación de registros erróneos a las capas de producción.
3. Procesamiento distribuido y privacidad de la información
La transformación de los datos validados se realiza mediante Apache Spark (PySpark). En esta etapa se ejecutan tareas de limpieza, deduplicación y enriquecimiento. En cumplimiento con normativas de protección de datos personales, se integra un módulo de enmascaramiento de PII (Personally Identifiable Information), aplicando técnicas de pseudonimización sobre columnas sensibles antes de su persistencia en la capa de producción.
4. Modelado dimensional con dbt
Para la estructuración analítica, se utiliza dbt (data build tool). Se ha desarrollado un modelo dimensional que organiza la información en tablas de hechos y dimensiones. El modelado sigue una progresión lógica desde vistas de staging hasta modelos marts de analítica avanzada, permitiendo una interpretación eficiente del negocio a través de métricas históricas y segmentaciones.
5. Orquestación y linaje
El flujo completo es coordinado por un único DAG en Apache Airflow, cumpliendo con el requisito de gestión centralizada. Al finalizar el proceso, se ejecuta una tarea de registro de linaje operacional que documenta la trazabilidad del dato desde su origen hasta su destino final, facilitando auditorías técnicas y el monitoreo de la frescura de la información.
Especificaciones tecnológicas
Orquestador: Apache Airflow 2.7.x
Motor de procesamiento: Apache Spark 3.3.0
Validación de calidad: Great Expectations 0.17.15
Transformación SQL: dbt 1.5.0
Base de Datos: PostgreSQL 15 (Warehouse analítico)
Infraestructura: Docker y Docker Compose


se usa el comando docker compose up -d --build para levantar el contendor
y para acceder a la interfaz de airflow la direccion es a: http://localhost:8080.
Y dentro de la interfaz se procede a activar y ejecutar el DAG dag_wap_unl_final.
Para la ejecución del proyecto, se requiere un entorno con Docker instalado y al menos 4GB de memoria RAM asignados.


Procedimiento de mantenimiento manual
En caso de requerir la reinicialización de esquemas y funciones de base de datos de manera manual, se debe ejecutar el siguiente procedimiento:
Despliegue del entorno

docker exec -it taller_etl_master-postgres_warehouse-1 psql -U user_dbt -d db_warehouse -c "
CREATE SCHEMA IF NOT EXISTS audit;
CREATE TABLE IF NOT EXISTS audit.data_freshness_monitor (id SERIAL, table_name TEXT, expected_arrival_time TIMESTAMP, actual_arrival_time TIMESTAMP, freshness_lag TEXT, status TEXT, threshold_minutes INT);
CREATE TABLE IF NOT EXISTS audit.data_lineage (lineage_id SERIAL, lineage_type TEXT, node_id TEXT, node_name TEXT, node_type TEXT, schema_name TEXT, table_name TEXT, description TEXT, source_id TEXT, target_id TEXT, transformation TEXT, dag_execution_id TEXT);
CREATE OR REPLACE FUNCTION public.pseudonymize_user(user_id INTEGER) RETURNS TEXT AS ' SELECT MD5(user_id::text || ''salt_unl_2026''); ' LANGUAGE SQL;


se realizó una intervención directa en la base de datos PostgreSQL mediante comandos docker exec, inicializando manualmente el esquema audit y sus tablas de monitoreo que no se crearon en el despliegue inicial. Asimismo, se habilitó la función pseudonymize_user requerida por dbt para el enmascaramiento de datos y se eliminaron tablas temporales huérfanas (__dbt_backup) que bloqueaban la creación de modelos en la capa de staging. Esta configuración manual desbloqueó el flujo orquestado por Airflow, permitiendo que el pipeline completara con éxito todas sus fases, garantizando la integridad de la validación y el registro final de linaje


logrando el flujo completo del ETL 
<img width="945" height="370" alt="image" src="https://github.com/user-attachments/assets/52068cf2-2adc-4397-8619-4e2f21e06d7f" />

