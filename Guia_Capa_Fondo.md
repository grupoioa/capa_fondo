# GUIA PARA GENERAR LA CAPA DE FONDO EN OWGIS  A PARTIR DE VARIAS IMAGENES .PNG

### Descarga de Imagenes
   Descargamos las 8 imagenes PNG del proyecto Blue Marble ( *[ir a la página][1]* ) :
   
   1. __world.topo.bathy.200412.3x21600x21600.A1.png__
   2. __world.topo.bathy.200412.3x21600x21600.B1.png__
   3. __world.topo.bathy.200412.3x21600x21600.C1.png__
   4. __world.topo.bathy.200412.3x21600x21600.D1.png__
   5. __world.topo.bathy.200412.3x21600x21600.A2.png__
   6. __world.topo.bathy.200412.3x21600x21600.B2.png__
   7. __world.topo.bathy.200412.3x21600x21600.C2.png__
   8. __world.topo.bathy.200412.3x21600x21600.D2.png__

-------------------------------

### Georeferenciar imagenes

   A cada imagen que descargamos hacemos la georefencia de coordenadas:  
   
   1. _**gdal_translate** -a_srs EPSG:4326 -a_ullr -180 90 -90  0 world.topo.bathy.200412.3x21600x21600.A1.png A1.tif_
   2. _**gdal_translate** -a_srs EPSG:4326 -a_ullr  -90 90   0  0 world.topo.bathy.200412.3x21600x21600.B1.png B1.tif_
   3. _**gdal_translate** -a_srs EPSG:4326 -a_ullr    0 90  90  0 world.topo.bathy.200412.3x21600x21600.C1.png C1.tif_
   4. _**gdal_translate** -a_srs EPSG:4326 -a_ullr   90 90 180  0 world.topo.bathy.200412.3x21600x21600.D1.png D1.tif_
   5. _**gdal_translate** -a_srs EPSG:4326 -a_ullr -180  0 -90 90 world.topo.bathy.200412.3x21600x21600.A2.png A2.tif_
   6. _**gdal_translate** -a_srs EPSG:4326 -a_ullr  -90  0   0 90 world.topo.bathy.200412.3x21600x21600.B2.png B2.tif_
   7. _**gdal_translate** -a_srs EPSG:4326 -a_ullr    0  0  90 90 world.topo.bathy.200412.3x21600x21600.C2.png C2.tif_
   8. _**gdal_translate** -a_srs EPSG:4326 -a_ullr   90  0 180 90 world.topo.bathy.200412.3x21600x21600.D2.png D2.tif_

------------------------------
### Mezclar los archivos .tif en un solo archivo .tif

   _**gdal_merge.py** -o capa_fondo.tif A1.tif B1.tif C1.tif D1.tif A2.tif B2.tif C2.tif D2.tif_
   
------------------------------
### Generamos la piramide de mosaicos

   _**gdal_retile.py** -v -r bilinear -levels 4 -ps 2048 2048 -co "TILED=YES" -co "COMPRESS=JPEG" -targetDir *directorio*    *capa_fondo.tif*_
  
************************************************************************************************************
# COMANDOS

### Como georeferenciar una imagen .PNG
#### Se tienen dos maneras para georeferenciar la imagen usando __gdal_translate__
1. Usando la bandera __-gcp__  
Lo que se hace es un mapeo de la posición del pixel a coordenadas _(x,y) -> (longitud,latitud)_

   Por ejemplo con una imagen de 5400 * 2400 :  
   __gdal_translate -gcp 0 0 -180 90 -gcp 5400 0 180 90 -gcp 0 2700 -180 -90 -gcp 5400 2700 180 -90 -a_srs EPSG:4326 imagen.png salida.tif__  
   * el pixel (0, 0)       sera en latitud/longitud el punto(-180, 90)
   * el pixel (5400, 0)    sera en latitud/longitud el punto(180, 90)
   * el pixel (0, 2700)    sera en latitud/longitud el punto(-180, -90)
   * el pixel (5400, 2700) sera en latitud/longitud el punto(180, -90)
   
2. Usando la bandera __-a_ullr__  
   Esta bandera unicamente necesita especificar a que las coordenas serán mapeadas los puntos de la esquina superior  
   izquierda y la esquina inferior derecha, por ejemplo en una imagen de 5400 * 2400:
   
   __gdal_translate -a_srs EPSG:4326 -a_ullr -180 90 180 -90 imagen.jpg out3.tif__
   
   en donde el mapeo de la esquina superior izquierda (ulx, uly) = (0, 0) se hace a la coordenada (-180, 90) y de la esquina      inferior derecha (lrx, lry) = (5400, 2400) se hace a la coordenada (180,-90).  
   * ulx = upper left x , uly = upper left y
   * lrx = lower right x, lry = lower right y  
   * _gdal_translate -a_srs tipo_proyeccion -a_ullr ulx uly lrx lry imagen.jpg out.tif_
   
-------------------------------
### Como unir (mezclar) varios archivos .tif

Por medio de la utilidad __gdal_merge.py__ que crea un mosaico de manera automatica a partir de un conjunto de imagenes (.tif), las cuales deben tener el mismo sistema de coordenadas. Se mezclan o unen el conjunto de imagenes que queremos manjear como una sola de la siguiente manera:

   __gdal_merge.py -o out_merge.tif file_1.tif file_2.tif . . . file_n.tif__

--------------------------------
### Como generar una piramide de mosaicos

Para generar la piramide de mosaicos lo haremos por medio de la utilidad __gdal_retile.py__ , por ejemplo :

__gdal_retile.py -v -r bilinear -levels 4 -ps 2048 2048 -co "TILED=YES" -co "COMPRESS=JPEG" -targetDir *directorio* *input.tif*__

[1]: https://visibleearth.nasa.gov/view.php?id=73909
