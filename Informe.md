# Proyecto de Satisfabilidad Booleana
CI-5437 - Inteligencia Artificial I, Universidad Simón Bolívar, abril - julio 2023

## Autores
- Rommel Llanos 15-10789
- Jesus Marcano 12-10359

## Resumen
Este proyecto tiene como objetivo modelar un problema en la Forma Normal Conjuntiva (FNC), utilizar un solucionador SAT para resolverlo y traducir la salida del solucionador SAT a un formato legible para humanos. Se pone especial énfasis en mantener transformaciones y modelos eficientes.

## Metodología:
El desarrollo de la solución a nuestro problema fue dividido en cuatro módulos de Python, disponibles en el repositorio del proyecto. Adicionalmente, creamos un script que permite unir estas funcionalidades en una cadena de procesos.

## Modelado del Problema:
En la etapa inicial, se realiza un preprocesamiento de los parámetros. Aquí se leen los parámetros de entrada y se calcula la cantidad de días hábiles disponibles para los partidos y la cantidad de bloques en los que puede ocurrir un partido, tomando en consideración las restricciones que un partido debe comenzar a una hora precisa y durar exactamente dos horas. Con estos parámetros en mano, la construcción de las cláusulas en el SAT que modela este problema se realiza de manera más sencilla. En caso de que los parámetros introducidos impidan la realización de partidos, se reporta como una entrada no válida.

### Modelado del Problema
En la primera etapa se ejecuta el preprocesamiento de los parámetros. Se leen los parámetros de entrada, se calcula el número de días laborables en los que pueden ocurrir los partidos y el número de bloques en los que puede tener lugar un partido. Si los parámetros dados impiden que se realicen los partidos, entonces se informa un mensaje de "entrada no válida".

### Formulación del Problema SAT
Nuestro enfoque para representar el problema consiste en una formulación SAT, donde inicialmente definimos literales que constituirán nuestras cláusulas. Estos literales se representan así:

x(i, j, d, b) = Representa que el equipo i está compitiendo en casa contra el equipo j (visitante) en el día 'd' durante el bloque de tiempo 'b'.

El objetivo es identificar cuáles de estos literales resultarán en un valor verdadero ('true') después de la ejecución del solucionador SAT.

Para garantizar el cumplimiento de las reglas del torneo, modelamos las siguientes condiciones:

Todos contra todos: Cada participante deberá enfrentarse dos veces a cada uno de los demás competidores, una vez como 'local' y la otra como 'visitante'. Para satisfacer este requerimiento, generamos los siguientes tipos de cláusulas:

Primero, se establece que cada par de equipos debe competir al menos una vez. Por lo tanto, para cada par de equipos i y j, generamos cláusulas compuestas por los literales x(i, j, d, b), donde 'd' y 'b' asumen todas las combinaciones posibles de días y bloques de tiempo.

Luego, se dictamina que cada par de equipos puede competir como máximo una vez. Para conseguir esto, para cada par de equipos i y j, generamos cláusulas (¬x(i, j, d, b) ∨ ¬x(i, j, d′, b′)) para todas las combinaciones distintas de días y bloques de tiempo 'd’ b’'. Esto modela la implicación de que si un equipo compite contra otro en un día y bloque de tiempo específicos, no podrá competir contra el mismo equipo en un día y bloque de tiempo diferentes.

Sin coincidencias simultáneas: No se pueden celebrar dos partidos al mismo tiempo. Para modelar esta restricción, generamos implicaciones que estipulan que si dos equipos juegan en un día y bloque de tiempo específicos, entonces cualquier otro par de equipos no puede competir ese mismo día y en ese mismo bloque de tiempo. Las cláusulas resultantes serían de la forma (¬x(i, j, d, b) ∨ ¬x(i′, j′, d, b)) para cada par de equipos i,j y cualquier otro par i',j'.

Un juego por día: Cada equipo solo puede jugar una vez al día. Para esto, generamos implicaciones que dictan que si dos equipos juegan en un día y bloque de tiempo específicos, cualquier otro tercer equipo no puede jugar contra estos equipos (ya sea en casa o de visita) en el mismo día. Adicionalmente, se agrega una implicación que impide que los mismos equipos compitan el mismo día, incluso si cambian de local.

Alternancia de localía: Cada participante no puede jugar como 'visitante' o 'local' dos días consecutivos. Para satisfacer esta condición, construimos cláusulas que representan la implicación de que si un par de equipos compite en un día y bloque de tiempo específicos, el equipo local no puede jugar como local contra cualquier otro equipo y en cualquier otro bloque de tiempo el día siguiente (lo mismo se aplica para el visitante). Las cláusulas se formulan de la siguiente manera: (¬x(i, j, d, b)∨¬x(i, k, d+1, b′)) y (¬x(i, j, d, b)∨¬x(k, j, d+1, b′)) para cualquier otro equipo k.

### Implementación
El proyecto fue implementado en Python. Para ejecutar el programa, primero instala el paquete `ics`, que se utilizará para crear archivos en formato iCalendar. Esto se puede hacer utilizando el siguiente comando en un entorno virtual: 
Ejecuta el comando:
```
 pip install -e <ruta-a-la-carpeta>/-proyecto-3-c15437. 
```
Por favor, verifica que estás utilizando la versión correcta de pip para tu intérprete de Python.

```
pip install -r requirements.txt
```
Genera el ejecutable de glucose ejecutando: 
```
make
```
Para generar el archivo que contiene el modelo del problema SAT en formato CNF DIMACS, ejecuta: 
```
python main.py --sat <input-json-file> <output-cnf-file>
```
Si quieres ejecutar todo el proceso, generando el archivo .ics en formato iCalendar, ejecuta:
```
python main.py --ical <input-json-file> <output-Ical-file>
```
Los módulos que implementan las funcionalidades son los siguientes:
- `preprocess.py`
- `sat_generator.py`
- `postprocess.py`
- `ical_generator.py`
- `main.py`

## Pruebas y Resultados
Diseñamos una serie de experimentos donde ejecutamos el programa con una serie de entradas, algunas con una posible solución y otras sin ella. A continuación, se muestra una tabla de comparación de la entrada y el rendimiento para cada uno de los casos ejecutados:

| Experimento | Participantes | Días | Bloques | Total de Cláusulas | Tiempo de Ejecución (s) | Tiempo de Glucose (s) | Solución Encontrada |
|-------------|---------------|------|---------|----------------------|-------------------------|-----------------------|---------------------|
| 1           | 7             | 15   | 5       | 754467              | 7.36                    | 2.85                  | Sí                 |
| 2           | 7             | 10   | 4       | 371742              | 43                      | 40.4                  | No                  |
| 3           | 9             | 20   | 5       | 221479              | 22.02                   | 20.74                 | Sí                 |
| 4           | 9             | 10   | 4       | 646632              | 35.19                   | 31.49                 | No                  |
| 5           | 9             | 30   | 12      | 22685256            | 33.97                   | 30.45                 | Sí                 |

A partir de estos resultados, podemos ver que el modelado del problema genera un número significativo de cláusulas, incluso para torneos pequeños. A pesar del gran número de cláusulas generadas, el solucionador SAT Glucose pudo resolver estos problemas dentro de límites de tiempo razonables. 

Sin embargo, a medida que aumenta el número de días, horas y participantes, el número de cláusulas generadas es tan grande que supera los recursos de memoria, lo que lo hace inadecuado para torneos más grandes. Por lo tanto, recomendamos usar este programa para torneos pequeños.
