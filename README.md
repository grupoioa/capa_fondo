# GUIA PARA GENERAR LA CAPA DE FONDO EN OWGIS A PARTIR DE IMAGENES .PNG
Esta guía es una descripción de una serie de comandos que nos ayudara a generar una capa `.tif` a partir de  
un conjunto de imganes `.png` (el caso es análogo para imagenes `.jpg`). En nuestro caso crearemos un archivo  
`.tif` que nos servira como una capa de fondo en OWGIS.  

## Primeros pasos
Lo que necesitaremos para replicar el ejemplo será las imagenes `.png`, descargamos las 8 imagenes del proyecto  
Blue Marble ( *[ir a la página][1]* ) :

   1. _world.topo.bathy.200412.3x21600x21600.A1.png_
   2. _world.topo.bathy.200412.3x21600x21600.B1.png_
   3. _world.topo.bathy.200412.3x21600x21600.C1.png_
   4. _world.topo.bathy.200412.3x21600x21600.D1.png_
   5. _world.topo.bathy.200412.3x21600x21600.A2.png_
   6. _world.topo.bathy.200412.3x21600x21600.B2.png_
   7. _world.topo.bathy.200412.3x21600x21600.C2.png_
   8. _world.topo.bathy.200412.3x21600x21600.D2.png_

### Prerrequisitos
Esto se realizó sobre Ubuntu 16.04 LTS utilizando `python3` con la biblioteca `GDAL`, las herramientas que se necesitan  
tener instaladas son :
  
   * _gdal_translate_
   * _gdal_merge.py_
   * _gdal_retile.py_

Estas herramientas se encuantran en la librería **GDAL**, para su instalación se puede hacer lo siguiente:  
   * Agregamos al repositorio y actualizamos:  
      `sudo add-apt-repository ppa:ubuntugis/ppa`  
      `sudo apt-get update`  
   
   * Ahora instalamos gdal y también las librerias para python:  
      `sudo apt-get install gdal-bin`  
      `sudo apt-get install libgdal-dev`  

#### Como georeferenciar una imagen .PNG
Se tienen dos maneras para georeferenciar la imagen usando `gdal_translate`
   1. Usando la bandera **-gcp**  
   Lo que se hace es un mapeo de la posición del pixel a coordenadas _(x,y) -> (longitud,latitud)_
   Por ejemplo con una imagen de 5400 * 2400 :  
    
   `gdal_translate -gcp 0 0 -180 90 -gcp 5400 0 180 90 -gcp 0 2700 -180 -90 -gcp 5400 2700 180 -90 -a_srs EPSG:4326 imagen.png salida.tif`
       * el pixel (0, 0)       sera en latitud/longitud el punto(-180, 90)
       * el pixel (5400, 0)    sera en latitud/longitud el punto(180, 90)
       * el pixel (0, 2700)    sera en latitud/longitud el punto(-180, -90)
       * el pixel (5400, 2700) sera en latitud/longitud el punto(180, -90)
   
   2. Usando la bandera __-a_ullr__  
   Esta bandera unicamente necesita especificar a que las coordenas serán mapeadas los puntos de la esquina superior  
   izquierda y la esquina inferior derecha, por ejemplo en una imagen de 5400 * 2400:
   
      `gdal_translate -a_srs EPSG:4326 -a_ullr -180 90 180 -90 imagen.jpg out3.tif`  
      
      en donde el mapeo de la esquina superior izquierda (ulx, uly) = (0, 0) se hace a la coordenada (-180, 90) y de la  
      esquina      inferior derecha (lrx, lry) = (5400, 2400) se hace a la coordenada (180,-90).  
         * ulx -> upper left x , uly -> upper left y  
         * lrx -> lower right x, lry -> lower right y  
         * _gdal_translate -a_srs tipo_proyeccion -a_ullr ulx uly lrx lry imagen.jpg out.tif_  

#### Como unir (mezclar) varios archivos .tif

Por medio de la utilidad __gdal_merge.py__ que crea un mosaico de manera automatica a partir de un conjunto de imagenes (.tif), las cuales deben tener el mismo sistema de coordenadas. Se mezclan o unen el conjunto de imagenes que queremos manjear como una sola de la siguiente manera:

   `gdal_merge.py -o out_merge.tif file_1.tif file_2.tif . . . file_n.tif`

#### Como generar una piramide de mosaicos

Para generar la piramide de mosaicos lo haremos por medio de la utilidad __gdal_retile.py__ , por ejemplo :

``gdal_retile.py -v -r bilinear -levels 4 -ps 2048 2048 -co "TILED=YES" -co "COMPRESS=JPEG" -targetDir directorio input.tif``

## Probando

### Georeferenciar imagenes

   A cada imagen que descargamos hacemos la georefencia de coordenadas:  
   
 ``1. gdal_translate -a_srs EPSG:4326 -a_ullr -180 90 -90  0 world.topo.bathy.200412.3x21600x21600.A1.png A1.tif  
   2. gdal_translate -a_srs EPSG:4326 -a_ullr  -90 90   0  0 world.topo.bathy.200412.3x21600x21600.B1.png B1.tif  
   3. gdal_translate -a_srs EPSG:4326 -a_ullr    0 90  90  0 world.topo.bathy.200412.3x21600x21600.C1.png C1.tif  
   4. gdal_translate -a_srs EPSG:4326 -a_ullr   90 90 180  0 world.topo.bathy.200412.3x21600x21600.D1.png D1.tif  
   5. gdal_translate -a_srs EPSG:4326 -a_ullr -180  0 -90 90 world.topo.bathy.200412.3x21600x21600.A2.png A2.tif  
   6. gdal_translate -a_srs EPSG:4326 -a_ullr  -90  0   0 90 world.topo.bathy.200412.3x21600x21600.B2.png B2.tif  
   7. gdal_translate -a_srs EPSG:4326 -a_ullr    0  0  90 90 world.topo.bathy.200412.3x21600x21600.C2.png C2.tif  
   8. gdal_translate -a_srs EPSG:4326 -a_ullr   90  0 180 90 world.topo.bathy.200412.3x21600x21600.D2.png D2.tif``

### Mezclar los archivos .tif en un solo archivo .tif

   `gdal_merge.py -o capa_fondo.tif A1.tif B1.tif C1.tif D1.tif A2.tif B2.tif C2.tif D2.tif`
   
### Generamos la piramide de mosaicos

   `gdal_retile.py -v -r bilinear -levels 4 -ps 2048 2048 -co "TILED=YES" -co "COMPRESS=JPEG" -targetDir directorio  
    capa_fondo.tif`

## Construido con
* [Python][2] Lenguaje de progración
* [GDAL][3] Librería usada.

## Versionando

Usamos [SemVer][4] para versionar. Para las versiones disponibles, consulte  
[las etiquetas en este repositorio][5].

## Autores
* **Raúl Medina Peña** - [Github][6]

## Licencia

Este proyecto está licenciado bajo la licencia MIT; consulte el archivo [LICENSE.md](LICENSE.md) para obtener detalles

## Agradecimientos

* Agradecemos a [Olmo Zavala][7] por todo su apoyo en este proyecto.

[1]: https://visibleearth.nasa.gov/view.php?id=73909
[2]: https://www.python.org/
[3]: https://www.gdal.org/
[4]: https://semver.org/lang/es/
[5]: https://github.com/your/project/tags
[6]: https://github.com/rmedina09
[7]: https://github.com/olmozavala
