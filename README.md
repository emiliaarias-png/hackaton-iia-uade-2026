# <M04> · <¿Este título va a la portada del usuario?>

**Integrantes:** <María Emilia Arias> · <Pilar Medina Livigni>
**IA declarada:** <Claude>
**Pista:** <Neurona>
**Publicación en LinkedIn:** <PONER EL LINK DE LINKEDIN>

---

## 1. Formulación y PEAS

**La pregunta.** <Para cada caso, (un par usuario - titulo candidato), ¿el título va a la portada del usuario? SÍ significa: el usuario mira más del 70% del título>

| PEAS | Su sistema |
|---|---|
| Medida de rendimiento | <Que los título que el sistema pone en portada sean los que el usuario termina de mirar (más del 70
%), medido con precisión y sensibilidad sobre los casos de prueba; a largo plazo, que el usuario siga en la plataforma.> |
| Entorno | <Plataforma de Streaming: la portada de cada usuario, su historial y el catálogo de títulos. Decide el recomendador de la portada.> |
| Actuadores | <Poner o no poner el título en la fila principal de la portada.> |
| Sensores | <Afinidad del título con su historial (0-1), duración (min) y calificación del título (0-1).> |

**Objetivo y métrica.** <El objetivo es que el usuario encuentre en su portada títulos que pueda disfrutar y terminar, para retenerlo a largo plazo. La métrica es qué tan bien coincide la respuesta del sistema con "mira más del 70%" (precisión y sensibilidad). No son lo mismo, y menos aún lo es el indicador que hoy premia el equipo de producto: la taza de clicks en la portada. Test práctico: ¿Cómo subir los clicks sin acercarse al objetivo? con miniaturas llamativas para títulos que el usuario no termina. En los datos de entrenamiento, los casos SI tienen promedio 10% de clicks esperados y los NO, 20% así que optimizar clicks empujaría para el lado contrario. Por eso no usamos "clicks esperados" ni como entrada ni como objetivo (ley de Goodhart: cuando una medida se vuelve objetivo, deja de ser buena medida).>

## 2. El sistema

**Cómo decide.** <Neurona (perceptrón con pesos y sesgo): calcula la entrada neta n = 10.x - 5,8, con x = afinidad(0-1). Responde SI si n => 0 (umbral u = 0), es decir, si la afinidad es 0,58 y NO si n < 0. Agente: El peso (10) y el sesgo (-5,8) se buscaron probando combinaciones con los casos 1-14(entrenamiento): el umbral de afinidad que acierta más casos de entrenamiento(12 de 14) es 0,58, justo en el medio entre el NO de afinidad 0,57, y el Si de afinidad 0,59. Probamos antes sistemas con tres y con dos variables, pero separaban el entrenamiento al 100 % (o con un solo tipo de error), lo que con solo 14 casos indica que el sistema se acomoda a esos casos y no aprende un patrón general. Por eso lo simplificamos a una variable. Aclaración honesta: durante la exploración también calculamos cómo salían algunos candidatos en los casos de prueba, pero la elección final se hizo por simplicidad y por los resultados de entrenamiento.>

**Variables que usa y por qué.** <Usa solo la afinidad del título con el historial del usuario, porque es la variable central de un recomendador. *De dónde sale y cuándo se calcula:* en la ficha viene como una columna ya calculada, de 0 a 1 por cada par usuario–título, y la ficha no explica la fórmula. Suponemos que la plataforma la calcula a partir del historial previo del usuario (lo que ya miró antes), *antes* de decidir si el título va a portada; con ese supuesto, no usa información del futuro. Si la afinidad incluyera lo que el usuario llegó a mirar de este mismo título, sería fuga de información y el sistema quedaría invalidado; por eso lo declaramos como supuesto. Descarta la calificación y la duración para limitar la complejidad y no sobreajustar, y los clics esperados de la miniatura porque es el indicador que premia el equipo de producto y no el objetivo (ver sección 1).>

**Cómo se corre en vivo.** <Cuaderno de Colab con la función "decidir(afinidad, duracion_min, calificacion)", que devuelve SI o NO: PENDIENTE(https://colab.research.google.com/drive/1pEcdVjE-lKZlqhQ8jpIE9WQW7r31HHsU?usp=sharing). El cuaderno también está en el repositorio como "sistema_M04.ipynb" se corre entero de arriba hacia abajo. A mano: afinidad × 10, menos 5,8; si da 0 o más, SÍ.>

*Dos casos de ejemplo.*

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | afinidad 0,80 → n = 8 − 5,8 = 2,2 | SÍ |
| B | afinidad 0,20 → n = 2 − 5,8 = −3,8 | NO |

## 3. Medición

*Entrenamiento (casos 1–14)*

| | Predice SÍ | Predice NO |
|---|---|---|
| *Real SÍ* | VP = 6 | FN = 1 |
| *Real NO* | FP = 1 | VN = 6 |

*Prueba (casos 15–20)*

| | Predice SÍ | Predice NO |
|---|---|---|
| *Real SÍ* | VP = 2 | FN = 0 |
| *Real NO* | FP = 1 | VN = 3 |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | 12/14 = 85,7 % | 5/6 = 83,3 % |
| Precisión | 6/7 = 85,7 % | 2/3 = 66,7 % |
| Sensibilidad | 6/7 = 85,7 % | 2/2 = 100 % |
| Especificidad | 6/7 = 85,7 % | 3/4 = 75 % |


**Entrenamiento contra prueba.** <Sí, hay diferencia: 33,3 puntos de exactitud(100% contra 66,7%). Es sobreajuste con 14 casos de entrenamiento y 4 números ajustables(3 pesos y el sesgo), el sistema se acomoda a esos casos y generaliza peor. En prueba, el caso 18(afinidad 0,34, calificación 9,3) es un Falso Positivo y el caso 19(afinidad al 0,98, calificación 3,3) es un Falso Negativo. Con solo 6 pasos de prueba, cada error mueve muchísimo las métricas, es por ello que son una señal débil.>

## 4. El error más caro

El falso positivo es el más caro: poner en portada un título que el usuario no termina desperdicia uno de los pocos lugares de la fila principal y le hace sentir al usuario que la plataforma no lo conoce, según la ficha. El falso negativo (dejar afuera algo que habría disfrutado) es una oportunidad perdida de retenerlo, pero no ocupa un lugar equivocado.

*Qué cambiamos en el sistema por ese error, y cómo lo medimos.* Cambiamos el umbral de afinidad: en lugar del valor por defecto de 0,50, usamos 0,58. La celda "El umbral y el error más caro" del cuaderno cuenta los errores de entrenamiento para varios umbrales:

| Umbral de afinidad | FP (entrenamiento) | FN (entrenamiento) | Costo con FP × 3 |
|---|---|---|---|
| 0,50 (por defecto) | 2 | 1 | 7 |
| *0,58 (elegido)* | *1* | *1* | *4* |
| 0,65 | 1 | 3 | 6 |
| 0,73 | 0 | 5 | 5 |

Pasar de 0,50 a 0,58 bajó los falsos positivos de entrenamiento de 2 a 1 sin sumar falsos negativos. Subir más el umbral (0,73) elimina el falso positivo pero deja 5 falsos negativos, que es peor aun pesando el falso positivo el triple. En los casos de prueba, el umbral 0,50 y el 0,58 dan la misma matriz, así que el cambio no altera los resultados de prueba. Además no usamos los clics esperados, que empujarían a poner títulos llamativos que no se terminan, y reportamos la precisión junto con la sensibilidad, no la exactitud sola.


## 5. ¿Comparado contra qué?

**Línea base trivial.** <Un sistema que responde siempre "NO" (la clase más frecuente en entrenamiento) saca 50% de exactitud en entrenamiento (7/14) y 66,7% en prueba(4/7), con sensibilidad 0%. Nuestra neurona empata en exactitud en prueba(66,7%), así que la exactitud sola no demuestra que sirva; por eso la métrica principal es la precisión(50%) junto con la sensibilidad(50%, contra 0% de la línea base: detecta títulos que sí se terminan). También comparamos con la regla que premia el equipo de producto ("SI si clics esperados => 15%"): da 28,6% de exactitud en entrenamiento y 33,3% en prueba, peor que la linea base.>

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | <20 casos inventados de la ficha (14 de entrenamiento y 6 de prueba); pocos y no son personas reales.> |
| ¿Qué métrica? | <Presición y sensibilidad; mostramos la exactitud pero siempre junto a la línea base.> |
| ¿Comparado contra qué? | <Contra responder siempre "NO" y contra la regla de clics.> |
| ¿Quién lo midió? | <Nosotras, con la división de entrenamiento y prueba fijada por la cátedra no hubo evaluación externa.> |

## 6. Riesgo: el camino completo

<Datos → modelo → decisión → consecuencia. *Datos:* 20 casos inventados, pocos y posiblemente no representativvos de los usuarios y del catálogo reales. *Modelo:* una neurona con 4 números ajustables que se sobreajusta a 14 casos. *Decisión:* SI si n => 0; el umbral fija quién absorbe cada error. *Consecuencia:* lo que se pone en portada es lo único que el usuario mira, así que genera los datos del ciclo siguiente (bucle de retroalimentación): lo que el sistema no muestra nunca produce datos sobre sí mismo. Riesgo concreto: optimizar un indicador que no es el objetivo(clics) llenaría la portada de títulos que no se terminan.>

## 7. Bitácora

*Uso de la IA*

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | Armar el sistema para la ficha M04 a partir de la ficha y las bases | Neurona con variables que se conocen antes de decidir; parámetros buscados con los casos de entrenamiento | Usar "clics esperados" como entrada: es el indicador que premia el equipo de producto y no el objetivo |
| 2 | Calcular las matrices de confusión y las métricas | Matrices que cierran con 14 y 6 casos; exactitud, precisión, sensibilidad y especificidad | Presentar la exactitud como métrica principal: es un número engañoso con pocos casos |
| 3 | Corregir el sistema porque el profesor indicó que no puede dar 100 % en entrenamiento ni en prueba | Simplificar la neurona a una sola variable (afinidad) | Mantener tres y dos variables: separaban el entrenamiento al 100 % o con un solo tipo de error, señal de sobreajuste. También rechazamos agregar casos a los datos: la ficha pide usar solo sus 20 casos |
| 4 | Mejorar el README según los agujeros del evaluador | Explicar de dónde sale la afinidad, medir el umbral con una tabla y agregar el análisis de firmeza de las métricas | Ninguno |
| 5 | Redactar el README con la plantilla de la cátedra | La estructura de las 7 secciones | Ninguno por ahora |

*Corridas contra el evaluador*

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | 8,5/10 (habilitadas para la mesa) | ¿De dónde sale la afinidad con el historial y en qué momento se calcula respecto de la decisión de poner el título en portada? Otras: si nunca se usaron clics, ¿qué cambió en el sistema para tener menos falsos positivos y se midió?; ¿qué pasa con las métricas de prueba si un solo caso cambia de lado? | Explicamos el origen de la afinidad y el supuesto de que se calcula antes de decidir; cambiamos el umbral de 0,50 a 0,58 y lo medimos en una tabla del cuaderno; agregamos el análisis de firmeza (un caso cambia de lado, intervalo de Wilson); completamos el link del Colab y esta bitácora |


**Integrantes:** <María Emilia Arias> · <Pilar Medina Livigni>
**IA declarada:** <Claude>
**Pista:** <Neurona>
**Publicación en LinkedIn:** <PONER EL LINK DE LINKEDIN>

---

## 1. Formulación y PEAS

**La pregunta.** <Para cada caso, ¿…? SÍ significa: …>

| PEAS | Su sistema |
|---|---|
| Medida de rendimiento | <cómo se sabe si decidió bien> |
| Entorno | <dónde y con qué decide> |
| Actuadores | <qué hace con la decisión> |
| Sensores | <qué datos recibe> |

**Objetivo y métrica.** <Qué es lo que de verdad importa en su problema y con qué métrica lo van a medir. ¿Son lo mismo?>

## 2. El sistema

**Cómo decide.** <Neurona: pesos y sesgo. Reglas: la lista de reglas si‑entonces. Agente: la regla con la que elige la acción.>

**Variables que usa y por qué.** <Cuáles usa, cuáles descarta y por qué.>

**Cómo se corre en vivo.** <Pasos concretos para resolver un caso nuevo: link al Colab, a la planilla o la cuenta a mano.>

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | <…> | <SÍ/NO> |
| B | <…> | <SÍ/NO> |

## 3. Medición

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <> | FN = <> |
| **Real NO** | FP = <> | VN = <> |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <> | FN = <> |
| **Real NO** | FP = <> | VN = <> |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | <> | <> |
| Precisión | <> | <> |
| Sensibilidad | <> | <> |
| Especificidad | <> | <> |

**Entrenamiento contra prueba.** <¿Hay diferencia? ¿Por qué? ¿Sobreajuste o generalización?>

## 4. El error más caro

<¿Es peor el falso positivo o el falso negativo en su problema? ¿Por qué, según la ficha? ¿Qué cambiaron en el sistema para cometer menos ese error?>

## 5. ¿Comparado contra qué?

**Línea base trivial.** <Qué resultado saca un sistema que responde siempre lo mismo, y cómo se compara el suyo.>

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | <> |
| ¿Qué métrica? | <> |
| ¿Comparado contra qué? | <> |
| ¿Quién lo midió? | <> |

## 6. Riesgo: el camino completo

<Datos → modelo → decisión → consecuencia aplicado a su caso. Al menos un riesgo concreto.>

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | <> | <> | <> |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | <> | <> | <> |
