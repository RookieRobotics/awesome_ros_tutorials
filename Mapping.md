# Mapeo 🗺️

> El proceso de mapeo empleado comúnmente se denomina SLAM (simultaneous localization and mapping por sus siglas en inglés) y se compone de algoritmos para la estimación de la posición y creación de mapas.

<br>

Actualmente existen diferentes paquetes en ROS para realizar tareas de mapeo. Por practicidad, en esta guía abordaremos el paquete llamado *[gmapping](https://wiki.ros.org/gmapping)* por su simplicidad y efectividad en entornos pequeños.
Para ejecutar el paquete se requiere un robot con odometría y un láser horizontal.

## Configuración inicial

> :memo: **Nota:** Para manipular el robot es necesario configurar un método de comunicación del tipo *master-slave*. El *master* representa la computadora del robot y es donde se publican los tópicos que provienen del robot. Los *slaves* son todas las computadoras que desean conectarse de manera remota al robot con el fin de suscribirse a sus tópicos.

Para configurar el *master*, tenemos dos alternativas para relizar la conexión: usar la red ethernet o wifi.

- Si usaremos una conexión ethernet es necesario saber la dirección ip de la computadora del robot. En este ejemplo asumiremos que la ip es 148.226.110.80. Tendremos que conectarnos a dicha ip mediante ssh:

```console
user@localhost:~$ ssh usuario@148.226.110.80
```

- Si usaremos una conexión wifi también es necesario conocer la dirección ip, ya que la red asigna una dirección diferente en ambos casos. Es necesario que la computadora *master* y la computadora *slave* se encuentren en la misma red wifi. En este ejemplo asumiremos que la ip es 192.168.0.102.

```console
user@localhost:~$ ssh usuario@ssh jhermosilla@192.168.0.102
```

Al entrar a la computadora del *master* es necesario exportar las siguientes variables en cada terminal que se utilice:

```console
user@localhost:~$ export ROS_MASTER_URI=http://ip_del_master:11311
user@localhost:~$ export ROS_IP=ip_del_master
```

<br>

> :bulb: **Tip:** Para apagar al robot (i.e., la computadora del *master*) de manera remota, se ejecuta el siguiente comando.

```console
user@localhost:~$ sudo shutdown -h now
```

## Recomendaciones antes de realizar el mapeo

> :warning: **Precaución:** Antes de realizar el mapeo, es recomendable asegurarnos que el lugar que escanearemos no contiene personas u objetos en movimiento ya que el escaner detectará estos movimientos.
 
Ya que se utiliza un laser horizontal es necesario considerar la altura del láser, debido a que sólo detectará los objetos a su altura, si hay objetos más pequeños o altos, no serán escaneados. Es recomendable adaptar el ambiente para que el láser pueda considerarlos. Por ejemplo, si tenemos una mesa de 4 bases, el láser sólo detectará las bases de la mesa, pero no el tablero en la parte superior, por lo que el espacio libre detectado debajo de la mesa no correspondería a su *footprint*. Para corregir esto se recomienda usar un mantel, de modo que el láser lo detecte como una pared y evite pasar por ahí.

> :warning: **Precaución:** Hay que tener cuidado con el vidrio. El láser traspasa el cristal, por lo que si hay algún vidrio o una puerta de cristal, no será considerada durante el mapeo, por lo que se recomienda cubrirlo o evitarlo.

Ya que el entorno puede cambiar, se recomienda marcar con cinta u otra alternativa la posición en el piso de los obstáculos que podrían moverse posterior al mapeo.

> :bulb: **Tip:** Deja todo en su lugar. Recuerda que al reorganizar el entorno, el mapa escaneado previamente no va a corresponder al entorno actual.

## Ejecución del paquete

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
