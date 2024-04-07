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
master@localhost:~$ export ROS_MASTER_URI=http://ip_del_master:11311
master@localhost:~$ export ROS_IP=ip_del_master
```

<br>

> :bulb: **Tip:** Para apagar al robot (i.e., la computadora del *master*) de manera remota, se ejecuta el siguiente comando.

```console
master@localhost:~$ sudo shutdown -h now
```

## Recomendaciones antes de realizar el mapeo

Ya que se utiliza un laser horizontal es necesario considerar la altura del láser, debido a que sólo detectará los objetos a su altura, si hay objetos más pequeños o altos, no serán escaneados.

> :warning: **Precaución:** Antes de realizar el mapeo, es recomendable asegurarnos que el lugar que escanearemos no contiene personas u objetos en movimiento ya que el escaner detectará estos movimientos.
 
Es recomendable adaptar el ambiente para que el láser pueda considerarlos. Por ejemplo, si tenemos una mesa de 4 bases, el láser sólo detectará las bases de la mesa, pero no el tablero en la parte superior, por lo que el espacio libre detectado debajo de la mesa no correspondería a su *footprint*. Para corregir esto se recomienda usar un mantel, de modo que el láser lo detecte como una pared y evite pasar por ahí.

> :warning: **Precaución:** Hay que tener cuidado con el vidrio. El láser traspasa el cristal, por lo que si hay algún vidrio o una puerta de cristal, no será considerada durante el mapeo, por lo que se recomienda cubrirlo o evitarlo.

Ya que el entorno puede cambiar, se recomienda marcar con cinta u otra alternativa la posición en el piso de los obstáculos que podrían moverse posterior al mapeo.

> :bulb: **Tip:** Deja todo en su lugar o marca tus obstáculos. Recuerda que al reorganizar el entorno, el mapa escaneado previamente podría no corresponder al entorno actual.

## Ejecución del paquete

Inicialmente, el paquete *gmapping* se puede ejectuar mediante el siguiente comando, sin embargo, es recomendable ajustar los parámetros disponibles según las carácterísticas del robot.

```console
master@localhost:~$ rosrun gmapping slam_gmapping scan:=base_scan
```

En este ejemplo, asumiremos que existe un launcher con la configuración necesaria para realizar el mapeo, esto se ejecuta mediante ssh en la computadora *master*:

```console
master@localhost:~$ roslaunch uverto uverto_mapping.launch
```

Esto lanzará el servicio de creación del mapa, para visualizar el mapeo se requiere que la computadora *slave* cuente con ROS y realizado la configuración previa. 

La visualización se realiza mediante *rviz* usando el tópico /map:

```console
slave@localhost:~$ rviz
```

> :warning: **Precaución:** Asegurarse que el fixed frame sea "map".

Una vez satisfecho con el mapa es necesario guardar el resultado en la computadora del *master*. Para guardar el mapa se ejecuta el siguiente comando en una terminal diferente:

```console
master@localhost:~$ rosrun map_server map_saver -f mymap
```

> :memo: **Nota:** La bandera -f nos ayuda a asignarle un nombre a nuestro mapa. Esto nos generará dos archivos, uno con extensión .pgm y otro con extensión .yaml 

## Cargar el mapa para tareas de navegación

Es necesario crear un paquete donde se utilizará el mapa generado. Una vez creado el paquete, crearemos un par de carpetas, una llamada *launch* y otra llamada *map*. En la carpeta map, debemos de colocar los archivos generados anteriormente.
En la carpeta *launch*, debemos lanzar el paquete mediante un archivo .launch que incluya la siguiente línea:

```yaml
<node pkg="map_server" type="map_server" name="map" output="screen" args="$(find demos_dia)/maps/firstmap.yaml"/>
```

Donde demos_dia es el paquete que creamos y firstmap.yaml es el arhcivo .yaml que generamos.

Este launcher es útil cuando se desean lanzar paquetes adicionales. Si usas un robot diferente, o tienes otro propósito es suficiente lanzar el paquete mediante la línea de comandos.

