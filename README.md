# EstudioSaludChile

Página web que muestra los resultados del estudio "Eficiencia en Pabellones quirúrgicos para Cirugías Electivas" de la Comisión Nacional de Productividad. Actualmente see encuentra en www.eficienciahospitalaria.cl.


## Estructura de archivos

Los archivos se encuentran organizados de la siguiente manera:

```
.
├─ public/
│   ├─ index.html          # Template html de la página, aquí se define el titulo de la página.
│   └─ favicon.png         # miniatura de la página.
│
├─ src/
│   ├─ assets/             # Archivos estáticos de la página.
│   │   ├─ data/           # Datos usados por los gráficos en formato JSON. Un archivo por vista.
│   │   ├─ fontawesome/    # Íconos usados en la página (más info en fontawesome.com).
│   │   ├─ img/            # Imágenes usadas (logo y gráficos)
│   │   ├─ maps/           # Mapas usados en formato GeoJSON.
│   │   └─ styles.css      # Estilos globales de la página.
│   │
│   ├─ common/             # Componentes comunes a todas las vistas: la barra de navegación y el footer.
│   │
│   ├─ components/         # Componentes utilizados por la aplicación, aquí se crean los gráficos.
│   │
│   ├─ views/              # Vistas de la aplicación, corresponde a las páginas que se muestran,
│   │                      # aquí se encuentran todos los textos de la página.
│   │
│   ├─ App.vue             # Define el template base de la página, se agrega el navbar y footer.
│   ├─ main.js             # importa las librerías y estilos usados en la página.
│   └─ router.js           # define las rutas por las cuáles se accederá a las distintas vistas.
│
├─ ...                     # archivos de configuración
```


## Componentes


## Instalar y ejecutar en local

Para intalar y ejecutar el proyecto de manera local se debe contar con `Node.js`.

``` bash
# instalar dependencias
npm install

# servir en localhost:8080 con recargado automático de los cambios
npm run serve
```


## Integración Continua

Este proyecto utiliza travis para realizar un paso a producción en Amazon Web Services (AWS) automático cada vez que se realizan cambios en el código. Para configurarlo se deben definir las siguientes variables de entorno en el proyecto en travis:

- `AWS_ACCESS_KEY_ID`: Id de la llave de acceso para AWS.
- `AWS_SECRET_ACCESS_KEY`: Llave secreta para AWS.
- `BUCKET_NAME`: Nombre del bucket de S3 en el que se guardará el proyecto.
- `BUCKET_REGION`: Región en la que se encuentra el bucket.
- `AWS_CLOUDFRONT_DIST_ID`: ID de la distribución de cloudfront.

* El proceso elimina todos los archivos existentes en el bucket antes de subir el código compilado del proyecto, por lo que no se debería guardar nada más ahí.
