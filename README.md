# Maquina expendedora

## Circuito
### Fritzing
![Fritzing](https://github.com/sandrag4/Maquina_expendedora-arduino/blob/main/fritzing_p3.jpg "Fritzing")


### Foto
![Foto](https://github.com/sandrag4/Maquina_expendedora-arduino/blob/main/circuito.jpeg "Foto")


## Video
[Video](https://urjc-my.sharepoint.com/:v:/g/personal/s_gonzaleza_2022_alumnos_urjc_es/EQgbeZ8Y791DswiIZuz30mQBf2PZg5pqB22cAmbV7NFMRA?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJPbmVEcml2ZUZvckJ1c2luZXNzIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXciLCJyZWZlcnJhbFZpZXciOiJNeUZpbGVzTGlua0NvcHkifX0&e=fiajIa)


## Threads
He creado los hilos arranque, servicio, admin. Cada hilo controla diferentes estados de la maquina de estados.
- arranque: Este hilo se habilita en el setup y ejecuta la funcion callback_arranque cada 1000 milisegundos. Cuando el LED ha parpadeado 3 veces, este hilo se inhabilita y se habilita el hilo de servicio.

- servicio: este hilo comienza inhabilitado y se habilita cuando el LED ha parpadeado 3 veces. Se vuelve a inhabilitar cuando la maquina expendedora pasa a modo admin, y se vuelve a habilitar cuando se vuelve a la funcionalidad de servicio. Cuando esta habilitado, ejecuta la funcion callback_servicio cada 300 milisegundos.

- admin: este hilo comienza inhabilitado y no se habilita hasta que se pasa al modo administrador. Se inhabilita de nuevo cuando vuelve a la funcionalidad de servicio. Cuando esta habilitado, ejecuta la funcion callback_admin cada 300 milisegundos.


## Maquina de estados
### Estados
He creado los estados: 
- ARRANQUE: Estado inicial de la maquina expendedora, se mantiene en este estado hasta que el led parpadea 3 veces, en ese momento el estado cambia a ESPERANDO_CLIENTE.

- ESPERANDO_CLIENTE: En este estado se obtiene la distancia al usuario mediante el sensor de ultrasonidos. Cuando el usuario se encuentra a menos de 1 metro, el estado pasa a TEMPERATURA. Si se pulsa el boton entre 2 y 3 segundos, se vuelve a este estado, y si se pulsa el boton durante mas de 5 segundo, si la maquina expendedora esta en modo administrador, tambien se vuelve a este estado.

- TEMPERATURA: Este estado muestra la temperatura y humedad obtenidas mediante el sensor DHT11. Cuando este estado esta en el hilo servicio, la temperatura y humedad se muestran durante 5 segundos y despues pasa al estado MENU_SERVICIO. Cuando este estado esta en el hilo admin, la temperatura y humedad se muestran hasta q el joystick se mueve hacia la izquierda, en ese momento el estado cambia a MENU_ADMIN.

- MENU_SERVICIO: En este estado se muestra el menu de seleccion de cafes. Se navega por la lista de cafes moviendo el joystick hacia arriba y hacia abajo. Cuando se pulse el joystick, se pasa al estado PREPARANDO_CAFE.

- PREPARANDO_CAFE: Este estado dura un tiempo aleatorio entre 4 y 8 segundos, un LED se va incrementado su intensidad. Cuando este estado finaliza, comienza el estado RETIRAR_BEBIDA.

- RETIRAR_BEBIDA: Se mantiene el estado RETIRAR_BEBIDA durante 3 segundos y despues se vuelve al estado ESPERANDO_CLIENTE.

- MENU_ADMIN: Este estado se activa si la maquina expendedora esta en la funcionalidad de servicio y se pulsa el boton durante mas de 5 segundos. En este estado se muestra el menu de administrador, se navega por la lista de funciones de administrador moviendo el joystick hacia arriba y hacia abajo. Cuando se pulse el joystick, se puede pasar a los estados TEMPERATURA, DISTANCIA, CONTADOR o LISTA_PRECIOS dependiendo de la funcionalidad elegida. Se vuelve a este estado moviendo el joystick hacia la izquierda.

- DISTANCIA: En este estado se muestra la distancia al usuario mediante el sensor de ultrasonidos. Este estado se mantiene hasta que se mueve el joystick hacia la izquierda.

- CONTADOR: Este estado muestra el tiempo en segundos que lleva la placa arrancada. Se sale de esta estado cuando se mueve el joystick hacia la izquierda.

- LISTA_PRECIOS: En este estado se muestra el menu de seleccion de cafes para elegir el cafe al que se quiere modificar el precio. Se navega por la lista de cafes moviendo el joystick hacia arriba y hacia abajo. Si se mueve el joystick hacia la izquierda, el estado vuelve a MENU_ADMIN. Cuando se pulse el joystick, se pasa al estado MODIFICAR_PRECIO.

- MODIFICAR_PRECIO: En este estado, se puede incrementar (moviendo el joystick hacia arriba)  o decrementar (moviendo el joystick hacia abajo) en 5 centimos el precio del cafe elegido. Si se pulsa el joystick, se guarda el precio establecido y se vuelve al estado LISTA_PRECIOS. Si se mueve el joystick hacia la izquierda, no se guarda el nuevo precio, y se vuelve al estado LISTA_PRECIOS.

 
### Hilos
- Estado del hilo arranque: ARRANQUE

- Estados del hilo servicio

  ```cpp
  void callback_servicio() {
    switch(state) {
        case ESPERANDO_CLIENTE:
            // ...
            break;
        case TEMPERATURA:
            // ...
            break;
        case MENU_SERVICIO:
            // ...
            break;
        case PREPARANDO_CAFE:      
            // ...
            break;
        case RETIRAR_BEBIDA:
            // ...
            break;
    }
  }
  ```

- Estados del hilo admin
  ```cpp
  void callback_admin() {
    switch(state) {
        case MENU_ADMIN:
            // ...
            break;
        case TEMPERATURA:
            // ...
            break;
        case DISTANCIA:
            // ...
            break;
        case CONTADOR:
            // ...
            break;
        case LISTA_PRECIOS:
            // ...
            break;
        case MODIFICAR_PRECIO:
            // ...
            break;
    }
  }
  ```


## Interrupciones
He configurado la interrupcion en CHANGE, la interrupcion salta cuando se presiona el boton o cuando se deja de presionar el boton. Si se empieza a presionar el boton, se guarda en una variable el tiempo actual usando millis(). Cuando se deja de presionar si el tiempo transcurrido desde que se comenzo a presionar el boton es usperior al tiempo antirrebote de 500 milisegundos, se comprueba si se ha presionado el boton durante un tiempo de entre 2 y 3 segundos o si se ha presionado durante mas de 5 segundos. Si se ha presionado entre 2 y 3 segundos, el estado cambia a ESPERANDO_CLIENTE. Si se ha presionado el boton durante mas de 5 segundos y estaba el hilo servicio activado, se encienden ambos LEDs y se pasa al estado MENU_ADMIN. Si se ha presionado el boton durante mas de 5 segundos y estaba el hilo servicio desactivado, se apagan ambos LEDs y el estado cambia a ESPERANDO_CLIENTE.


## Watchdog
He contigurado el watchdog para que se reinicie cada 2 segundos.