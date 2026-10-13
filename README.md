
Proyecto: ReLooper : sampler/looper/sequencer

[Link al video de presentación:](https://drive.google.com/file/d/1t9QU4xkxeqvmIcVS6_HWj957HrwJK3w3)

[Link al código](https://drive.google.com/drive/folders/1cKuk-sbbddEl7YKP4t0HP-5ucop5rYKe)

[Link a la carpeta del patch completo con ejemplos y samples](https://drive.google.com/drive/folders/1VL5NR4QDwOPUFznvQELv-rhdbrRSvd8P)

- [Presentacion ([YouTube](https://youtube.com/vi/2I8aFGHPuAw/0.jpg))](https://youtu.be/2I8aFGHPuAw)
  
- [Ejemplo 1 - Sgt Pepper RMX (YouTube)](https://youtu.be/laUeQBn42l4)
  
- [Ejemplo 2 - 2 segundos (YouTube)](https://youtu.be/0zD6zCd_ow8)


Introducción:

ReLooper es un patch desarrollado en Pure Data que implementa un looper que trabaja con
samples de audio y permite secuenciar parámetros con el objetivo performático de manipular y
reformular sonidos en tiempo real. Está pensado para ofrecer un acceso sencillo, directo y
experimental que permita una transformación radical del material tras poco tiempo de uso, con
una curva de aprendizaje suave.

Trabaja con cuatro pistas simultáneas. Cada pista puede cargar un archivo de audio, trozarlo en
partes iguales o pasos, reordenarlos, silenciarlos e invertirlos, también modificar su altura (pitch),
modular su paneo y su volumen mediante osciladores de baja frecuencia y filtrarlos. Todas las
pistas llegan a un master común que las suma, y alimenta tanto la salida de audio del sistema
como un grabador.

La elección de las variables a modular tuvo objetivo de lograr la mayor transformación posible. Se
eligió trabajar sobre los parámetros mas representativos de las muestras de audio como el tono o
pitch, el silencio, la duración, el sentido de la reproducción, y el orden interno.
No posee otros efectos de audio, excepto filtros, trémolo y paneo, con el fin de mejorar la
espacialidad y facilitar coexistencia de las distintas pistas en la mezcla final.

Arquitectura

ReLooper presenta un diseño modular: un patch principal (Reloop.pd) instancia varios subpatches
y cuatro pistas/canales idénticos en su forma, conformados por múltiples abstracciones
reutilizables, cada una encapsulada en su propio archivo .pd y parametrizada con $1 = 1, 2, 3 o 4
en el momento de su creación.

Cada una de las cuatro pistas incluye un subpatch bang y un array que muestra la forma de onda
del archivo de audio (TrackN). El subpatch bang dispara la carga del archivo al recibir un bang
desde Cargar audio, o desde selector_pasos cuando se selecciona un nuevo número de pasos
(pasosN); a continuación, un mensaje read carga el archivo wav correspondiente.

Luego, cada pista se compone de varias abstracciones de control “por paso” (controlGate,
controlOrden, controlPitch, controlRepeat, controlVol y una instancia de controlMute), más el motor
de reproducción controlPlay. Este diseño separa cada dimensión paramétrica de la pista en un
módulo propio, lo que favorece la legibilidad del patch raíz y permite visualizar y razonar de forma
aislada sobre cada dimensión.

Los módulos controlOrden, controlRepeat, controlPitch y controlVol comparten una estructura casi
idéntica: un array de 16 posiciones, cuyo eje horizontal representa el índice de reproducción
indice$1 que avanza de a uno tanto pasos como lo determine el “selector de pasos”y vuelve a
empezar, luego el eje vertical representa el valor del parámetro a modular para tal índice, con un
rango propio de cada módulo, más un selector de modo que rescribe o reordena el array. En el
interior de cada módulo tenemos una lista de 17 elementos ("pack 0 0 1 2 ... 16") enviada por send
al array correspondiente ($1).

Los modos de escritura disponibles son: “random 1”, que asigna valores aleatorios a cada paso y
lo podemos disparar pulsando el selector; “random 2”, similar al anterior, pero que vuelve a
generarlos en forma cíclica, cada un tiempo en milisegundos expresado en el número “ciclo
random 2”; se inicializa con la duración total del audio dur$1. “Dibujar”, que permite introducir los
valores “a mano”. “Selector” o “Slider” en controlPitch y controlVol, que permite elegir un valor para
que permanezca invariable ante el avance del índice de reproducción; y “Ordenar”, exclusivo de
controlOrden, que vuelve al orden de reproducción normal del audio. Ver figuras 1, 7, 8, 9 y 10.
A la derecha de la pantalla se encuentran tres subpatches comunes: transporte, master y
selector_pasos.

Flujo de la señal de audio

El recorrido de la señal de cada pista empieza con la carga del archivo mediante soundfiler en
controlPlay, el motor de reproducción. La señal pasa luego por pitchNuevo, que realiza la
transposición; por controlVol, que aplica el volumen por paso, el trémolo y el paneo; y por
masterFiltro, con una instancia por pista. Finalmente, Master suma las cuatro pistas y envía el
resultado a dac~ y al grabador (REC). 

Subpatches globales

Pd Transporte
Control de transporte global, carga de archivos, play/stop, grabación, y reset global (I/O) de las grillas de
dieciséis pasos que habilita o silencia cada paso. Ahi dentro se encuentran los subpatches REC, botonPlay,
refrescar y cargar.
Obs.: cada nueva grabación que se haga dentro de una misma sesión va a estar numerada (grabación 1,
grabación 2, etc.) y se va a imprimir en consola. Al terminar una sesión se debe tener la precaución de
almacenar los audios grabados para que no sean reescritos en la siguiente sesión. 


Pd Master
Suma de las cuatro pistas y control de volumen general, filtro con control de frecuencia de corte y selector
de tipo de filtrado LP/HP por track, y salida a dac~ y al grabador. El seteo por defecto es lpf y frecuencia de
corte en 15000 Hz. 

Pd selector_pasos
Selector de pasos y subdivisiones por track. Selector compartido que define la cantidad de pasos
por pista y las envía por send pasos1, 2, 3 y 4 hacia los subpatches bang1, 2, 3 y 4. El número de
pasos por defecto para todos los tracks es dieciséis y se muestra en el número al final de cada
selector. 
Obs. el número elegido no modifica el largo total del audio, solo afecta la cantidad y duración de
cada paso. Este subpatch tiene gran influencia en todo el patch. Es clave para generar poliritmias,
11 golpes contra 9, por ejemplo. O silencios inesperados al no encontrar tal paso en el array
orden$1. En “repeticiones” cambia el valor de la subdivisión, y en “pitch” puede afectar con una
sola variación tramos de audio más extensos.


Subpatches por pista

Pd bangN
Recibe un bang desde selector_pasos para poner en marcha la carga del archivo wav.

Pd playTrack
Recibe el play/stop global pero también actúa de forma local para poner en marcha o detener la
pista a voluntad. Se encuentra dentro de controlPlay.pd.

Pd envolvente
Se activa por pasos. Su principal objetivo es evitar los clics que se escuchan debido al corte y
reordenamiento de cada tramo de audio. Recibe desde controlVol los valores de las variables
atk$1 y rel$1 dados por sendos sliders para controlar el ataque y el release de la envolvente. Se
encuentra dentro de controlPlay.pd y se dispara con cada paso del metro principal.
En la práctica se permiten valores de hasta 200 ms para el ataque porque generan efectos más
audibles, como, por ejemplo, imprimir carácter rítmico a un tono continuo tipo drone.

Pd infoConsola
Imprime en consola la información de cada archivo propia de cada track al momento de cargar los
audios. Recibe iniciar.

Abstracciones

controlPlay.pd – Motor de audio
Es el módulo mas importante y el de mayor complejidad interna. Recibe por su inlet el mensaje de
lectura y, mediante soundfiler, carga el archivo de audio en la tabla Track$1. Con el número total
de muestras que devuelve soundfiler calcula la duración del archivo en milisegundos
(durSample$1), que también se usa como valor inicial del ciclo de random 2 (metroRandom2$1);
el tamaño de cada paso en muestras (muestrasPaso$1), dividiendo el total por la cantidad de
pasos (pasos$1); y la duración de cada paso en milisegundos (dur$1). Como la cantidad de pasos
entra por un inlet frío, estos valores se recalculan recién al volver a cargar el archivo, tarea de la
que se encarga el subpatch bang del patch raíz.
El reloj de la pista es un metro con período dur$1, que se pone en marcha o se detiene desde Pd
playTrack. En cada disparo envía pasoMetro$1, activa la envolvente e incrementa en uno un
contador. Una operación de módulo limitada por pasos$1 convierte ese contador en el índice de
paso (indice$1), que vuelve a 0 después del último paso y hace posible la reproducción en loop. Al
recibir stop, el contador se reinicia en 0.
El índice recorre el array orden$1 de controlOrden, que indica qué tramo de audio corresponde a
cada paso. Ese número de tramo, multiplicado por muestrasPaso$1, da la posición de inicio de
lectura dentro de la tabla (posRead$1). La reproducción del audio se realiza mediante dos
motores que funcionan en paralelo: tabplay~, que lee el tramo al derecho a partir de posRead$1, y
tabread4~, que lo lee al revés. Para la lectura en reversa, en cada paso un line~ salta al final del
tramo (posRead$1 + muestrasPaso$1) y recorre una rampa hasta su inicio en dur$1 milisegundos,
de modo que el tramo se lee hacia atrás a la misma velocidad que en la reproducción normal. El
toggle Normal/Reverse, controlado por reverse$1 desde controlOrden, multiplica cada motor por 1
o por 0, para que solo uno de los dos llegue a la salida.
También incorpora un efecto de repetición, que divide cada paso en fragmentos aún más
pequeños. Un segundo metro, con un período de dur$1 / repeDiv$1, vuelve a disparar la lectura
desde el inicio del tramo, tanto al derecho como al revés, y pone en loop ese fragmento hasta que
llega el paso siguiente. El valor de repeDiv$1 llega desde el selector derecho de controlRepeat.
Un condicional enciende este metro solo cuando el play y las repeticiones están activos a la vez,
lo que evita arranques o paradas involuntarias de la lectura al recibir el el bang desde play/stop.
La suma de ambos motores se multiplica por la envolvente descrita en Pd envolvente y luego por
el valor de mute$1, enviado por la instancia de controlMute. La señal resultante de cada pista se
envía al bus de mezcla mediante send de audio (s~ bus$1).
Visualmente contamos con control de transporte con botones individuales de play, de mute, I/O
para refrescar control Gate, más dos ventanas con información sobre la cantidad de muestras del
archivo cargado y su duración total en milisegundos. Además, el subpatch pd infoConsola imprime
en la consola, al recibir el mensaje iniciar, el número de muestras, la duración del archivo, el
número de pasos, las muestras por paso y la duración de cada paso. Ver Figura 5.
Obs.: las conversiones de muestras a milisegundos usan una frecuencia de muestreo fija de
44100 Hz, por lo que los archivos deben estar a esa frecuencia para que las duraciones
calculadas coincidan con las reales.

controlGate.pd – Gate por pasos
Grilla de 16 pasos que habilita o silencia cada paso de la secuencia de la pista N, con resaltado
visual sincronizado al índice de reproducción. Sus 16 toggles arman el valor de gate$1, que se
usa en controlVol para silenciar pasos puntuales sin tocar el resto de los parámetros. Está
conectada a los controles I/O de controlPlay y transporte. 

controlOrden.pd – Orden de reproducción
Selector de reproducción por paso. Cuenta con un botón para activar un modo de reproducción en
reverse (envía reverse$1 a controlPlay) y un array de 16 x 16, cuyo eje vertical muestra el tramo
de audio que está siendo reproducido, de 1 a 16.

controlRepeat.pd – Repeticiones
Selector de repetición por paso; Presenta un botón para activar el efecto, un array de 16 x 7 y un
selector de modos de reproducción. El eje vertical representa en cuántas partes está subdividido
el tramo de audio, con siete opciones: potencias de 2, de 1 a 64.

controlPitch.pd – Transposición de tono
Transpone el pitch por pasos sin modificar su velocidad y alimenta la abstracción pitchNuevo. Ver .
El eje vertical representa la variación de pitch, con un rango de 0.5 a 4 (de 1/2 * f a 4 * f): tres
octavas, una hacia abajo y dos hacia arriba.

controlVol.pd – Control de volumen y paneo
Control de volumen por paso, es un array de dieciséis pasos con un rango de 0 a 1 en el eje
vertical. Además, cuenta con funciones que modulan la amplitud y el paneo de manera más
general: un trémolo, que se activa desde el mismo botón de apagado de efectos presente en los
otros arrays con dos sliders verticales que controlan frecuencia (fr) e intensidad (int). Luego
presenta un slider horizontal “grande” para un control manual del paneo, un botón bang para
volver el paneo al centro, y la posibilidad de activar un paneo automático por LFO de frecuencia e
intensidad ajustables con los dos sliders ubicados abajo a la izquierda.
Los slider abajo a la derecha ajustan el ataque y el release de la envolvente por pasos (ver Pd
envolvente);
Obs: si accionamos el trémolo, los valores del array no surten efecto alguno.

– masterFiltro.pd: filtro estéreo con selector hpf o lpf, descrito en Pd Master. Vive dentro de ese
subpatch.

– tremolo.pd y autopan.pd: LFO que modulan la amplitud y el paneo de la pista,
respectivamente. Viven dentro de la abstracción controlVol.pd.

– pitchNuevo.pd: pitch shifter por línea de retardo variable que ejecuta la transposición de tono,
recibe el valor desde controlPitch.

– randomizer16.pd / randomizer16_float.pd / randomizer16_1001.pd: Tres variantes de un
mismo generador de 16 valores aleatorios, con distinto rango de salida según el parámetro que
alimentan (orden/repetición, pitch, volumen).


