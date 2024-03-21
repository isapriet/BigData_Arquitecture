# Diseño del DAaaS

Definición la estrategia del DAaaS.
Plataforma de newsletter personalizada que proporciona a los suscriptores una selección semanal de las mejores propiedades inmobiliarias para invertir. Esta frecuencia de envío puede ser efectiva para mantener a los usuarios comprometidos sin saturar sus bandejas de entrada. La plataforma está diseñada para satisfacer las necesidades de una audiencia diversa interesada en la inversión inmobiliaria, ya sea a largo plazo, por habitaciones o para alquiler vacacional. Se incluyen descripciones detalladas, url y datos clave como el precio, ubicación o número de habitaciones.
El usuario podrá escoger sus preferencias y modificarlas en cualquier momento según sus intereses.
Además de brindar un servicio valioso a los usuarios, esta plataforma puede permitir monetizar a través de ofertas premium para suscriptores.
Utilización de herramientas de análisis para medir métricas clave como tasas de apertura, tasas de clics y tasas de conversión.
El mercado inmobiliario es uno de los más sólidos y rentables. Las personas buscan activamente oportunidades de inversión, la calidad de los datos, la automatización y la segmentación permite ofrecer una experiencia de usuario excepcional.

# Arquitectura DAaaS

1. Fuentes de datos:
■ Api de idealista.com en la que podemos buscar inmuebles utilizando el siguiente ejemplo:
curl -X POST -H "Authorization: Bearer {{OAUTH_BEARER}}" -H "Content-Type: multipart/form-data;" -F "center=40.430,-3.702" -F "propertyType=homes" -F "distance=15000" -F "operation=sale" "https: //api.idealista.com/3.5/es/search"
■ Crawling para avisar de nuevos inmuebles publicados y scraping para extraer los datos de las siguientes web:
  ○ Fotocasa: https://www.fotocasa.es/es/
  ○ Milanuncios: https://www.milanuncios.com/inmobiliaria/
  ○ Habitaclia: https://www.habitaclia.com/
■ Airbnb dataset de información de inmuebles vacacionales en España.

2. Componentes:
■ Google Compute Engine: Máquina virtual en donde estarán alojados la web y MongoDB.
■ Web estática: Crea un formulario con las peticiones del usuario y sus datos.
■ MongoDB: Instancia de MongoDB en la nube para guardar los formularios.
■ Google BigQuery: Procesar y analizar los datos (recomendable SQL para
mejorar el rendimiento en el procesado).
■ Google Cloud SQL: Herramienta para guardar los datos limpios y listos para
mostrar.
■ Scheduler: Ejecución automática de trabajos.
■ Cloud Storage: Almacenamiento de PDF listos para enviar al usuario.
■ Google Cloud SendGrid: Envío del mail al usuario.
■ Cloud Functions:
  ○ Request a la API de Idealista.com y formatear los datos según requisitos del modelo de datos.
  ○ Crawling y Scraping de las webs inmobiliarias (fotocasa, milanuncios y Habitaclia).
  ○ Petición a Mongodb para saber que datos y preferencias tiene el usuario y modificar las tablas creadas por Google BigQuery que son mandadas al almacenamiento de Google Cloud SQL.
  ○ Recibe datos de Google Cloud SQL, los sube al Cloud Storage y cuando estén listos, baja el pdf y mediante Google Cloud SendGrid manda el mail con el resultado al usuario.

# DAaaS Operating Model Design and Rollout

Crear Scheduler una vez a la semana que lanza Cloud Functions para recoger datos de diferentes portales inmobiliarios y en base a un formulario previo del usuario que ha rellenado en nuestra web, analizamos el mercado en busca de las preferencias que nos ha indicado el usuario, generando un PDF con enlaces que mandamos por mail en forma de Newsletter.

1. Creación de un proyecto en Google Cloud.
   
2. Google Compute Engine: creamos una instancia de VM Linux en Compute Engine:
  ■ Web Estática: Crea una página web estática con un formulario donde los usuarios pueden ingresar sus preferencias para la compra de inmuebles. Recolecta información como ubicación, tipo de inmueble (residencial, comercial, etc.), presupuesto, preferencias de tamaño, características
  específicas, etc.
  ■ MongoDB en la nube: Utilizamos una instancia de MongoDB en la nube para
  almacenar los datos de los formularios completados por los usuarios. Cada registro en la base de datos debe estar asociado con un usuario y sus preferencias.

3. Google Cloud Functions:
  ■ Crea cloud functions en python para realizar diversas tareas
    i. Request a la API de idealista.com: Un Cloud Function puede realizar peticiones a la API de Idealista.com utilizando la librería nativa de Python, Request, para obtener información actualizada sobre inmuebles en función de las preferencias de los usuarios. Los datos
    se envían a una instancia de BigQuery para su posterior análisis.
    ii. Crawling y Scraping de Webs Inmobiliarias: Utilizamos 3 Cloud Functions que realizan el crawling y scraping de sitios web inmobiliarios como Fotocasa, Milanuncios y Habitaclia. Utilizando BeautifulSoup y scrapy. La información obtenida se envía a una
    instancia de BigQuery.
    iii. Petición a MongoDB de los formularios y formatear los datos que
    queremos comunicar entre BigQuery y Google Cloud SQL.
    iv. Generador de PDF con los datos que recibimos ya filtrados de Google Cloud SQL utilizar la librería Aspose de Python e integramos el servicio de Google SendGrid para generar el mail que enviamos al
    usuario.

4. Google BigQuery para el análisis de datos:
  ■ Configuración de BigQuery: En primer lugar, configurar un conjunto de datos
  en BigQuery. Se puede crear uno a través del Panel de Control de Google
  Cloud o utilizando la interfaz de línea de comandos de Google Cloud.
  ■ Creación de Tablas: Crear las tablas necesarias para el análisis de datos. Se cargan los datos en estas tablas desde Cloud Functions o desde otras fuentes de datos utilizando las herramientas proporcionadas por Google
  Cloud.
  ■ Creación de consultas SQL avanzadas. Los resultados de las consultas se
  envían directamente a un CLoud Function que formateando el response
  almacenará los resultados en Google Cloud SQL.
  ■ Importar los datos de Airbnb.csv.

5. Google Cloud SQL:
  ■ Configurar una instancia de Google Cloud SQL en PostgreSQL, creando la
  base de datos y tablas personalizadas para almacenar los datos procesados.
  ■ Desde tus Cloud Functions, configura una conexión a la instancia de Google Cloud SQL utilizando las credenciales adecuadas. Puedes utilizar bibliotecas o SDKs para el lenguaje de programación que estés utilizando en tus Cloud
  Functions.

6. Cloud Storage: Crear Bucket para que una vez generados los PDFs personalizados, se almacenen en Google Cloud Storage para su posterior acceso y distribución.
