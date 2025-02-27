Euskal Encounter 21 - AI Contest
================================================================================
                                 FAROS LÁSER
================================================================================

v0.7

NOTA: esta especificación está sujeta a cambio. En particular, se reserva el
derecho de cambiar los valores numéricos mencionados, que no son definitivos.

================================================================================
El Mapa
================================================================================

El juego se jugará en un tablero de dos dimensiones compuesto por una matriz de
casillas. Cada partida transcurrirá sobre un mapa de forma y dimensiones fijadas
al comienzo de ella. El mapa define qué subconjunto de las casillas forma el
área jugable (la isla). En todos los casos las casillas del borde del mapa no
formarán parte del área jugable (lo cual excluye la posibilidad de mapas con
"wraparound"). Además, todas las casillas de la "isla" estarán conectadas entre
sí (no son válidos los mapas con áreas disjuntas).

Ejemplo de un mapa válido:

XXXXXXXXXXXX
XXXX       X
X      XX  X
XX  X     XX
XX  XXXX  XX
XX        XX
XXXXXXXXXXXX

Se considera que la casilla inferior izquierda es la casilla (0,0) y las
coordenadas crecen hacia arriba y a la derecha.

================================================================================
Energía
================================================================================

La energía es indispensable para jugar el juego. Cada casilla de la isla tendrá
una cantidad de energía disponible, inicialmente 0, que se irá incrementando
durante la partida. Los jugadores que pisan una casilla obtienen la energía
que hay en ella, que pasa de nuevo a 0. La energía siempre es un entero (no hay
unidades fraccionarias).

================================================================================
Los jugadores
================================================================================

Cada partida la jugarán dos o más jugadores identificados por los números 0, 1,
2, etc.

Cada jugador tiene una posición actual en el mapa. La posición al comienzo de la
partida estará definida en el mapa.

Los jugadores podrán viajar por la isla de forma libre, en cualquiera de las
ocho direcciones (horizontal, vertical, y diagonal). Varios jugadores pueden
ocupar la misma casilla en el mismo momento. No se podrá transcurrir por las
casillas que no forman parte de la isla.

Los jugadores dispondrán de una reserva de energía (0 al comienzo de la
partida). La energía se obtendrá de las casillas. En principio no habrá límite
sobre la energía de cada jugador, aunque se reserva el derecho a establecer un
máximo.

================================================================================
Los faros
================================================================================

En el mapa se situarán una serie de faros. Cada faro puede ser neutro o estar
controlado por un jugador. Cuando un faro está controlado por un jugador, tendrá
una cantidad de energía asociada a el. Esta energía disminuirá con el tiempo,
y cuando llega a 0 el faro pasará a ser neutro de nuevo. No hay un máximo de
energía por faro.

Los faros generan energía de forma continua en las casillas cercanas a
ellos. Esta energía no proviene del faro en sí, es decir, es independiente de
si el faro está controlado por un jugador, o si es neutro, y de su nivel actual
de energía.

================================================================================
Conexiones
================================================================================

Los jugadores podrán conectar parejas de faros mediante haces láser cuando ambos
están bajo su control. Sin embargo, está prohibido lanzar un haz entre dos faros
que cruce un haz ya existente (tanto del mismo jugador como del enemigo). Si se
hiciera, la densidad de flujo lumínico en la intersección sería tan alta que se
iniciaría una fusión no controlada de la atmósfera y por lo tanto una catástrofe
global. También está prohibido conectar dos faros si la conexión pasaría
exactamente por un tercer faro.

Para realizar una conexión entre dos faros se debe estar en la casilla de uno
de ellos y tener la clave del otro faro. Para obtener la clave de un faro se
deberá visitarlo, y sólo se puede tener una clave en cada momento, que se
perderá al realizar una conexión (las claves son de un sólo uso, OTP).

Cuando tres faros se conectan diréctamente entre sí formando un triángulo, se
iluminan todas las casillas situadas en el interior de dicho triángulo. El
objetivo del juego es tener el máximo número de casillas iluminadas por
áreas triangulares entre faros bajo el control del mismo jugador.

Las conexiones pueden cruzar casillas que no forman parte de la isla. Las
casillas que no forman parte de la isla no se iluminan ni puntúan, pero pueden
formar parte de un triángulo.

================================================================================
El Juego
================================================================================

La partida constará de una serie de rondas. Durante cada ronda primero será el
turno del jugador 0, después el 1, etc.

Al comienzo del juego, cada jugador recibirá la siguiente información:
- Su número de jugador
- El número de jugadores total
- Su posición inicial
- El mapa (es decir un mapa booleano de qué casillas forman parte de la isla)
- Por cada faro:
  - Sus coordenadas

Al comienzo de cada ronda ocurrirá lo siguiente:
- Cada casilla incrementará su cantidad de energía libre disponible según la
  siguiente fórmula:
    energía += floor(5 - distancia a faro)
  Esto ocurre por cada faro, de forma que las casillas a menos de 5 unidades
  de distancia lineal a más de un faro se verán afectadas por todos ellos.
  Cada casilla se limitará  a un máximo de 100 unidades de energía.
- Cada jugador obtendrá de su casilla actual la cantidad de energía libre
  presente en ella, que se reducirá a cero. Si varios jugadores comparten
  casilla, cada uno recibirá la fracción correspondiente (si hay resto tras la
  división, la energía restante se pierde).
- Si un jugador está situado sobre un faro, obtendrá su clave si no dispone ya
  de ella. Esto ocurre indistintamente de si el faro está controlado por el o
  no.
- La energía de todos los faros se decrementa en 10 puntos. Si llega a cero,
  el faro pasa a ser neutro y desaparecen todas las conexiones con él.

Antes de cada turno, el jugador recibe la siguiente información:

- Su posición
- Su puntuación
- Su nivel de energía acumulada
- Por cada casilla cuyo centro se haya a 3 unidades o menos del jugador (en
  línea recta):
  - La cantidad de energía libre disponible en ella. Lógicamente, el valor
    siempre será cero para la casilla del propio jugador, ya que esa energía ya
    ya sido obtenida.
- Por cada faro:
  - Sus coordenadas (como identificador)
  - El jugador que lo controla (0, 1, 2, ... o ninguno=-1)
  - Su nivel de energía (cero si no está bajo control)
  - La lista de faros a los cuales está conectado (por lo tanto cada conexión se
    indicará por duplicado, desde ambos vértices).
  - Si se dispone de su clave o no.

Cabe resaltar que los jugadores NO conocen la posición de los demás.

Cada turno, cada jugador podrá realizar UNA de las siguientes acciones:

- Nada
- Mover a una casilla adyacente
- Si se encuentra en la misma casilla que un faro:
  - Atacar o recargarlo. Atacar o recargar un faro es la misma acción: el
    jugador aporta una cantidad de energía de su elección. Si el faro no está
    bajo su control, se resta total o parcialmente de la energía presente en el
    faro. Cuando toda la energía del faro ha sido eliminada, el sobrante de
    energía aportada se suma de nuevo al faro y el faro pasa a estar controlado
    por el jugador actual.
  - Conectarlo a otro faro bajo el control del mismo jugador. Para ello, el
    jugador debe disponer de la clave del faro remoto, y la perderá al crear
    la conexión.


================================================================================
Puntuación
================================================================================

Cada jugador tendrá una puntuación (0 al comienzo de la partida), que es un
número entero. Se obtendrán puntos al término de cada ronda de la siguiente
forma:

- 2 puntos por cada torre bajo control del jugador.
- 2 puntos por cada pareja de faros conectados bajo el control del jugador.
- Por cada 3 faros conectados entre sí bajo su control (formando un triángulo):
  - 1 punto por cada casilla cuyo centro esté dentro de dicho triángulo.

  Nota 1: es posible que las áreas se solapen, y, en ese caso, puntúan doble.
  Por ejemplo, en esta configuración de faros conectados existen 3 triángulos
  interiores y un triángulo exterior, de forma que la cantidad de puntos
  otorgados será equivalente al DOBLE del área del triángulo exterior.

  o-----o
   \`o'/
    \|/     
     o

  Igualmente, es posible que un área de un jugador contenga completamente a
  un área de otro jugador. En este caso cada jugador obtiene sus puntos
  correspondientes. Es decir, la casilla se puede "iluminar" de una mezcla de
  colores.

  Nota 2: Cuando un lado del triángulo pasa por el centro de una casilla, se
  utilizará la regla de rasterización de OpenGL para determinar qué casillas
  se consideran como parte del área (cuentan las casillas superiores y a la
  izquierda): http://goo.gl/f1cxU

================================================================================
Fin del juego
================================================================================

El juego terminará tras un número predefinido de rondas.

================================================================================
Protocolo
================================================================================

Los bots se comunicarán por stdio/stdout. El protocolo consiste en mensajes
JSON codificados en una sola línea, terminados por \n (newline). El motor del
juego siempre es el responsable de iniciar las comunicaciones, y el bot
contestará con su respuesta. Los bots pueden mostrar mensajes de debug por
stderr, pero DEBEN prefijarlos por su nombre entre [] para su clara
identificación:

[TroloBot] mensaje...

Nota: los siguiente ejemplos están formateados en varias líneas, pero en
el protocolo real deberán estar contenidos exclusivamente en una línea.

----------------
Inicio del juego
----------------
El motor envía el siguiente mensaje (ejemplo):
{
	"player_num": 0,
	"player_count": 2,
	"position": [1, 2],
	"map": [
		[0, 0, 0, 0, 0],
		[0, 1, 1, 1, 0],
		[0, 1, 1, 0, 0],
		[0, 1, 1, 0, 0],
		[0, 0, 0, 0, 0]],
	"lighthouses": [
		[1, 1], [3, 1], [2, 3], [1, 3]
	]
}

Este mensaje indica que el bot es el primer jugador (jugador 0) de 2 en total
(el otro sería el jugador 1), y describe el siguiente mapa de 5x5:

v----- casilla (0, 4)
XXXXX<-- casilla (4,4)
X!!XX
X0 XX
X! !X
XXXXX<-- casilla (4,0)
^----- casilla (0,0)

! - faro
0 - posición inicial del jugador 0

Nótese que el mapa se envía de abajo hacia arriba (0,0 es la primera casilla
que se envía, pero es la esquina inferior izquierda). En la práctica la
dirección en la que se visualice el mapa es inconsecuente para el juego,
excepto en lo que respecta a la iluminación de casillas, que sigue la regla
de bordes izquierdo y superior asumiendo que (0,0) es la esquina inferior
izquierda.

El bot contesta con el siguiente mensaje:
{
	"name": "TroloBot"
}

El nombre se utilizará para mostrar el nombre del bot en pantalla.
El bot debe inicializarse y contestar en un máximo de 2 segundos tras el
envío del mensaje de inicio.

----------------
Turno
----------------
El motor envía el siguiente mensaje con el estado actual (ejemplo):
{
	"position": [1, 3],
	"score": 36,
	"energy": 66,
	"view": [
		[-1,-1,-1, 0,-1,-1,-1],
		[-1, 0, 0,50,23,50,-1],
		[-1, 0, 0,32,41, 0,-1],
		[ 0, 0, 0, 0,50, 0, 0],
		[-1, 0, 0, 0, 0, 0,-1],
		[-1, 0, 0, 0, 0, 0,-1],
		[-1,-1,-1, 0,-1,-1,-1]
	],
	"lighthouses": [
		{
			"position": [1, 1],
			"owner": 0,
			"energy": 30,
			"connections": [[1, 3]],
			"have_key": false
		},
		{
			"position": [3, 1],
			"owner": -1,
			"energy": 0,
			"connections": [],
			"have_key": false
		},
		{
			"position": [2, 3],
			"owner": 1,
			"energy": 90,
			"connections": [],
			"have_key": false
		},
		{
			"position": [1, 3],
			"owner": 0,
			"energy": 50,
			"connections": [[1, 1]],
			"have_key": true
		}
	]
}

"view" es un mapa de la energía disponible en las celdas cercanas al jugador.
Nominalmente es de 7x7, conteniendo las casillas a distancia 3 o menos del
jugador. Los bots deben tolerar que este valor varíe, pero siempre se dará
el caso de que "view" es una matriz cuadrada de dimensión impar, y la posición
del bot corresponde a la casilla del medio de la matriz. Las celdas fuera del
radio 3 se devuelven como -1 para indicar que no hay datos. Las celdas que
no forman parte de la isla se devuelven como 0 ya que nunca pueden contener
energía (las casillas que sí forman parte de la isla pero no tienen energía
también se devuelven como 0). Ya que el jugador obtiene la energía de la
casilla central al comienzo de cada ronda, esta siempre tendrá un valor de 0.

En este caso, la vista nos proporciona la siguiente información sobre el mapa:

X  X  X  X ??
X () 50  X  X
X 32 41  X ??
X 50 23 50 ??
?? X ?? ?? ??

() - posición jugador 0 (energía 0)
?? - no hay datos (en este mapa, por ser pequeño, resulta que todas estas
casillas no forman parte de la isla; este no sería el caso en mapas mayores)
X - casilla no forma parte de la isla, pero entra dentro del área visible
    (distancia 3 o menor). Este hecho viene del mapa del mensaje inicial.

El mensaje describe los faros siguientes:

X  X  X  X  X
X 50 90  X  X 
X  |     X  X 
X 30     0  X 
X  X  X  X  X 

Donde cada número indica la energía del faro. Los faros con energía == 50, 30
están controlados por el jugador 0 y conectados entre sí, y el faro con
energía == 90 está controlado por el jugador 1. Bajo estas condiciones, el
jugador 0 obtiene 6 puntos por ronda, y el jugador 1 obtiene 2 por ronda.
Además, el jugador 0 tiene la clave del faro con energía 50 (es de suponer que
visitó el faro 30 para obtener su clave, y luego el 50 para conectarlo al
30, perdiendo la clave del 30 en el proceso, pero ahora dispone de la clave del
50 por estar situado sobre el).

El bot debe contestar con uno de los siguientes mensajes (tiempo máximo por
turno: 100ms desde que se recibe el estado hasta que se contesta):
{
	"command": "pass"
}
No hacer nada (pasar el turno)

{
	"command": "move",
	"x": -1,
	"y": 1
}
Mover a una casilla adyacente (en este caso, arriba y hacia la izquierda).
Obviamente x e y deben ser -1, 0, o 1. Mover 0,0 es equivalente a pasar el
turno.

{
	"command": "attack",
	"energy": 80
}
Atacar o recargar el faro sobre el cual se encuentra el jugador. Por ejemplo,
si el faro está controlado por otro jugador con energía 50, entonces pasaría a
estar controlado por el jugador actual con energía 30. Si el faro está
controlado por otro jugador con energía 90, entonces seguiría controlado por el
mismo jugador con energía 10. Si el faro está controlado por el jugador actual
con energía 40, entonces pasa a tener energía 120. Si el faro está controlado
por otro jugador con energía 80, entonces pasa a ser neutro (no controlado por
ningún jugador) con energía 0. Al perderse o cambiar el control de un faro,
se destruyen todas las conexiones actuales y por extensión dejan de existir
todas las áreas (triángulos) que tienen ese faro como vértice (aunque es posible
que un área que contenga íntegramente al faro no se vea afectada). La energía
suministrada debe ser igual o menor a la energía que posee el jugador
actualmente (si no lo fuera, se limita automáticamente al total disponible).
Atacar con energía 0 es equivalente a pasar el turno.

{
	"command": "connect",
	"destination": [1, 1]
}
Conectar el faro de la posición actual al faro (1, 1). El jugador debe poseer
la clave del faro de destino, y ambos deben estar controlados por el jugador.
La conexión no debe existir ya, ni cruzarse contra ninguna otra conexión
existente, ni pasar por el centro de un tercer faro (la conexión se puede
solapar con la casilla de un tercer faro siempre y cuando no lo haga por su
centro exacto).

El motor del juego contesta con el resultado de la operación:
{
	"success": true
}
O bien (ejemplo de error):
{
	"success": false,
	"message": "Player does not have the destination key"
}
En caso de error, el resultado es el mismo que si el jugador hubiera pasado el
turno.

----------------
Fin del juego
----------------
Al término del juego, el motor simplemente cierra stdin y stdout. El bot debe
cerrarse correctamente al detectar una condición de EOF en stdin.


En resumen, las comunicaciones con el bot siguen la siguiente progresión:
<< Inicialización (mapa, faros, etc.)
>> Hello (nombre del bot)
<< Estado al comienzo del turno (faros, conexiones, energía, etc.)
>> Jugada (comando)
<< Resultado de la jugada
<< Estado al comienzo del turno (faros, conexiones, energía, etc.)
>> Jugada (comando)
<< Resultado de la jugada
(...)
<< Estado al comienzo del turno (faros, conexiones, energía, etc.)
>> Jugada (comando)
<< Resultado de la jugada
<< [EOF]
>> [EOF]

