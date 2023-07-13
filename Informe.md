# Proyecto de Satisfabilidad Booleana
CI-5437 - Inteligencia Artificial I, Universidad Simón Bolívar, abril - julio 2023

## Autores
- Rommel Llanos 15-10789
- Jesus Marcano 12-10359

## Resumen
Este proyecto tiene como objetivo modelar un problema en la Forma Normal Conjuntiva (FNC), utilizar un solucionador SAT para resolverlo y traducir la salida del solucionador SAT a un formato legible para humanos. Se pone especial énfasis en mantener transformaciones y modelos eficientes.

## Enfoque
La solución se desglosó en cuatro módulos de Python, todos disponibles en el repositorio del proyecto. También creamos un script para unir las diversas funcionalidades.

### Modelado del Problema
En la primera etapa se ejecuta el preprocesamiento de los parámetros. Se leen los parámetros de entrada, se calcula el número de días laborables en los que pueden ocurrir los partidos y el número de bloques en los que puede tener lugar un partido. Si los parámetros dados impiden que se realicen los partidos, entonces se informa un mensaje de "entrada no válida".

### Modelado SAT
El problema se modela como un problema SAT utilizando literales definidos de la siguiente manera: 
```
x(i, j, d, b) = El equipo i juega en casa vs. el equipo j (visitante) en el día d en el bloque b
```
Esto reduce el problema a encontrar cuáles de estos literales tendrán el valor 'verdadero' al final de la ejecución del solucionador SAT.

Para hacer cumplir las reglas del torneo, modelamos lo siguiente:
- **Regla de todos contra todos:** Cada participante deberá jugar dos veces con cada uno de los demás participantes, una vez como "visitantes" y la otra como "locales".
- **Regla de no juegos concurrentes:** Dos partidos no pueden ocurrir al mismo tiempo.
- **Regla de una vez al día:** Un participante puede jugar como máximo una vez por día.
- **Regla de cambio de ubicación:** Un participante no puede jugar como "visitante" en dos días consecutivos, ni como "local" en dos días consecutivos.

### Implementación
El proyecto fue implementado en Python. Para ejecutar el programa, primero instala el paquete `ics`, que se utilizará para crear archivos en formato iCalendar. Esto se puede hacer utilizando el siguiente comando en un entorno virtual: 
Ejecuta el comando:
```
 pip install -e <ruta-a-la-carpeta>/proyecto-3-c15437-main. 
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