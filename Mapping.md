# Mapeo 🗺️

> El proceso de mapeo empleado comúnmente se denomina SLAM (simultaneous localization and mapping por sus siglas en inglés) y se compone de algoritmos para la estimación de la posición y creación de mapas de costos.

<br>

Actualmente existen diferentes paquetes en ROS para realizar tareas de mapeo. Por practicidad, en esta guía abordaremos el paquete llamado *[gmapping](https://wiki.ros.org/gmapping)* por su simplicidad y efectividad en entornos pequeños.
Para ejecutar el paquete se requiere un robot con odometría y un láser horizontal.

## Configuración inicial

TODO: Describir el proceso de conexión al master y exportación de ip's.
    //Por ethernet 
        Uvetro tiene por default la ip 148.226.110.80

        ssh jhermosilla@148.226.110.80
        Esto solicitará una contraseña: iiia2024


    //Por wifi
        Verificar que la nuc y la computadora esclavo con la cual se comunicará se encuentren en la misma red wifi.
        Verificar la Ip de la nuc (ya que es dinámica) y conectarse a la ip de la nuc.
        En este caso, la ip ha sido 192.168.0.102

        ssh jhermosilla@192.168.0.102
        Esto solicitará una contraseña: iiia2024

        realizar los siguiente export en cada terminal que se utilice:

        export ROS_MASTER_URI=http://ipMaster:11311
        export ROS_IP=ipMaster

        Lo cual quedaría de la siguiente manera con la IP dada:
        export ROS_MASTER_URI=http://192.168.0.102:11311
        export ROS_IP=192.168.0.102

    NOTA: Para apagar al robot (la nuc) de manera remota, ejecutar el siguiente comando:
    sudo shutdown -h now

## Recomendaciones para realizar el mapeo
> Antes de realizar el mapeo, debemos de asegurarnos que el lugar que escanearemos deb de encontrarse de preferencia sin personas u objetos en movimiento.
Esto debido a que ek escaner detectará estos movimientos.
> Considerar la altura del láser. Si utilizará un laser horizontal, hay que tener en cuenta que sólo detectará los objetos a su altura, si hjay objetos más bajos 
  o más altos, no los podrá visualizar, por lo que habá que adaptarlos para que el láser pueda verlos. Por ejemplo, si tenemos una mesa de 4 patas, el láser sólo detectará las paras de la mesa, pero no la altura de la mesa, por lo que puede detecat como espacio libre debajo de la mesa, lo cual no sería lo ideal. Para corregir esto, podríamos ponerle un mantel a la mesa para que el láser lo detecte como una pared y evite pasar por ahí.
> Cuidado con el vidrio. El láser traspasa el cristal, por lo que si hay algún vidrio o una puerta de cristal, el robot no la va a detectar, por lo que igual se recomienda recurbirla. 
> Deja todo en su lugar. Recuerda que si después de realizar el mapeo, reorganizan el entorno, el robot tendrá guardado el mapa de cómo estaba acomodado anteriormente, por lo que si se mueve algo, se tenría que realizar nuevamente el mapeo.

## Ejecutar el launcher para crear el mapa

Una vez dentro de uberto, ejetucar el siguiente launcher:
roslaunch uverto uverto_mapping.launch
Esto lanzará el servicio de creación del mapa, para verificar lo que se va detectando, ejecutar rviz:
rviz
Asegurarse de que rviz tenga las siguientes configuraciones:
Fixed frame: map
añadir(add) el mapa en el tópico correspondiente: /map

Para guardar el mapa generado, realizar lo siguiente en otra terminal en la misma compiutadora donde se ejecutó el launcher:
rosrun map_server map_saver -f mymap
La bandera -f nos ayuda a asignarle un nombre a nuestro mapa.

Esto nos generará dos archivos, uno con extensión .pgm y otro con extensión .yaml 


## Cargar el mapa en un launcher 
Crear un paquete, el cual utilizará el mapa generado.
Una vez creado el paquete, crearemos un par de carpetas, una llamda launch y otra llamada map
En la carpeta map, debemos de colocar los mapas generados anteriormente.
En la carpeta launch del siguiente archivo, debemos agregar lo siguiente:
/* TODO: Agregar contenido del launcher */

Este launcher sólo servirá siemre y cuando se tengan los paquetes necesarios, los cuales ya tiene cargados nuestro robot.
Si estas usando un robot diferente, este archivo no te servirá, sólamente la línea donde se importa el mapa, la cual es la siguiente:

<node pkg="map_server" type="map_server" name="map" output="screen" args="$(find demos_dia)/maps/firstmap.yaml"/>

Donde demos_dia es el paquete que creamos y firstmap.yaml es el arhcivo .yaml que generamos.



## Ejecución

TOOD: Describir la ejecución del launcher, guardado del mapa y recomendaciones durante el mapeo.

## Visualización

TODO: Describir la configuración necesaria para visualizar el mapa en Rviz.
