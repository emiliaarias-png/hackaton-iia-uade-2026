# M04 · ¿Este título va a la portada del usuario?

**Integrantes:** Pilar Medina Livigni · María Emilia Arias
**IA declarada:** Claude
**Pista:** Neurona
**Publicación en LinkedIn:** PENDIENTE (se publica al terminar el hackatón)

---

## 1. Formulación y PEAS

**La pregunta.** Para cada caso (un par usuario–título candidato), ¿el título va a la portada del usuario? SÍ significa: el usuario mira más del 70 % del título.

| PEAS | Su sistema |
|---|---|
| Medida de rendimiento | Que los títulos que el sistema pone en portada sean los que el usuario termina de mirar (más del 70 %), medido con precisión y sensibilidad sobre los casos de prueba; a largo plazo, que el usuario siga en la plataforma. |
| Entorno | Plataforma de streaming: la portada de cada usuario, su historial y el catálogo de títulos. Decide el recomendador de la portada. |
| Actuadores | Poner o no poner el título en la fila principal de la portada. |
| Sensores | Afinidad del título con el historial del usuario (0 a 1). |

**Objetivo y métrica.** El objetivo es que el usuario encuentre en su portada títulos que disfruta y termina, para retenerlo a largo plazo. La métrica es qué tan bien coincide la respuesta del sistema con "mira más del 70 %" (precisión y sensibilidad). No son lo mismo, y menos aún lo es el indicador que hoy premia el equipo de producto: la tasa de clics en la portada. Test práctico: ¿cómo subir los clics sin acercarse al objetivo? Con miniaturas llamativas para títulos que el usuario no termina. En los datos de entrenamiento los casos SÍ tienen en promedio 10 % de clics esperados y los NO, 20 %, así que optimizar clics empujaría para el lado contrario. Por eso no usamos "clics esperados" ni como entrada ni como objetivo (ley de Goodhart: cuando una medida se vuelve objetivo, deja de ser buena medida).

## 2. El sistema

**Cómo decide.** Es una neurona de una sola entrada (perceptrón con peso y sesgo). Calcula la entrada neta n = 10·x − 5,8, con x = afinidad (0 a 1). Responde SÍ si n ≥ 0 (umbral u = 0), es decir, si la afinidad es 0,58 o más, y NO si n < 0. El peso (10) y el sesgo (−5,8) se buscaron con los casos 1 a 14 (entrenamiento): el umbral de afinidad que acierta más casos de entrenamiento (12 de 14) es 0,58, justo en el medio entre el NO de afinidad 0,57 y el SÍ de afinidad 0,59. Probamos antes sistemas con tres y con dos variables, pero separaban el entrenamiento al 100 % (o con un solo tipo de error), lo que con solo 14 casos indica que el sistema se acomoda a esos casos y no aprende un patrón general. Por eso lo simplificamos a una variable. Aclaración honesta: durante la exploración también calculamos cómo salían algunos candidatos en los casos de prueba, pero la elección final se hizo por simplicidad y por los resultados de entrenamiento.

**Variables que usa y por qué.** Usa solo la afinidad del título con el historial del usuario, porque es la variable central de un recomendador. **De dónde sale y cuándo se calcula:** en la ficha viene como una columna ya calculada, de 0 a 1 por cada par usuario–título, y la ficha no explica la fórmula. Suponemos que la plataforma la calcula a partir del historial previo del usuario (lo que ya miró antes), **antes** de decidir si el título va a portada; con ese supuesto, no usa información del futuro. Si la afinidad incluyera lo que el usuario llegó a mirar de este mismo título, sería fuga de información y el sistema quedaría invalidado; por eso lo declaramos como supuesto. Descarta la calificación y la duración para limitar la complejidad y no sobreajustar, y los clics esperados de la miniatura porque es el indicador que premia el equipo de producto y no el objetivo (ver sección 1).

**Cómo se corre en vivo.** Cuaderno de Colab con la función `decidir(afinidad)`, que devuelve SÍ o NO: https://colab.research.google.com/drive/1pEcdVjE-lKZlqhQ8jpIE9WQW7r31HHsU?usp=sharing (acceso de lectura). El cuaderno también está en el repositorio como `sistema_M04.ipynb`; se corre entero de arriba hacia abajo. A mano: afinidad × 10, menos 5,8; si da 0 o más, SÍ.

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | afinidad 0,80 → n = 8 − 5,8 = 2,2 | SÍ |
| B | afinidad 0,20 → n = 2 − 5,8 = −3,8 | NO |

## 3. Medición

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 6 | FN = 1 |
| **Real NO** | FP = 1 | VN = 6 |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 2 | FN = 0 |
| **Real NO** | FP = 1 | VN = 3 |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | 12/14 = 85,7 % | 5/6 = 83,3 % |
| Precisión | 6/7 = 85,7 % | 2/3 = 66,7 % |
| Sensibilidad | 6/7 = 85,7 % | 2/2 = 100 % |
| Especificidad | 6/7 = 85,7 % | 3/4 = 75 % |

**Entrenamiento contra prueba.** La diferencia de exactitud es chica: 2,4 puntos (85,7 % contra 83,3 %), así que no hay una señal fuerte de sobreajuste: el sistema es simple y rinde parecido con casos que no vio. La precisión baja de 85,7 % a 66,7 %. En entrenamiento el falso negativo es el caso 4 (afinidad 0,26, pero se miró más del 70 %) y el falso positivo es el caso 14 (afinidad 0,71, pero no se terminó). En prueba el falso positivo es el caso 20 (afinidad 0,71). La sensibilidad de prueba da 100 % porque los dos casos SÍ (15 y 19) tienen afinidad alta (0,83 y 0,98), pero con solo 2 casos SÍ es poca evidencia.

**Qué tan firmes son las métricas de prueba.** Con 6 casos, cada caso vale 16,7 puntos de exactitud. Si un solo caso de prueba cambiara de lado, la exactitud quedaría entre 66,7 % y 100 %, y la precisión entre 50 % y 100 %. El intervalo aproximado (Wilson, 95 %) para la exactitud de prueba va de 44 % a 97 %. Entonces el 83,3 % no es firme: indica que el sistema no es peor que la línea base, pero no alcanza para afirmar cuánto mejor es. Esto está calculado en una celda del cuaderno.

## 4. El error más caro

El falso positivo es el más caro: poner en portada un título que el usuario no termina desperdicia uno de los pocos lugares de la fila principal y le hace sentir al usuario que la plataforma no lo conoce, según la ficha. El falso negativo (dejar afuera algo que habría disfrutado) es una oportunidad perdida de retenerlo, pero no ocupa un lugar equivocado.

**Qué cambiamos en el sistema por ese error, y cómo lo medimos.** Cambiamos el umbral de afinidad: en lugar del valor por defecto de 0,50, usamos 0,58. La celda "El umbral y el error más caro" del cuaderno cuenta los errores de entrenamiento para varios umbrales:

| Umbral de afinidad | FP (entrenamiento) | FN (entrenamiento) | Costo con FP × 3 |
|---|---|---|---|
| 0,50 (por defecto) | 2 | 1 | 7 |
| **0,58 (elegido)** | **1** | **1** | **4** |
| 0,65 | 1 | 3 | 6 |
| 0,73 | 0 | 5 | 5 |

Pasar de 0,50 a 0,58 bajó los falsos positivos de entrenamiento de 2 a 1 sin sumar falsos negativos. Subir más el umbral (0,73) elimina el falso positivo pero deja 5 falsos negativos, que es peor aun pesando el falso positivo el triple. En los casos de prueba, el umbral 0,50 y el 0,58 dan la misma matriz, así que el cambio no altera los resultados de prueba. Además no usamos los clics esperados, que empujarían a poner títulos llamativos que no se terminan, y reportamos la precisión junto con la sensibilidad, no la exactitud sola.

## 5. ¿Comparado contra qué?

**Línea base trivial.** Un sistema que responde siempre "NO" (la clase más frecuente en entrenamiento) saca 50 % de exactitud en entrenamiento (7/14) y 66,7 % en prueba (4/6), con sensibilidad 0 %. Nuestra neurona lo supera: 85,7 % en entrenamiento y 83,3 % en prueba, con sensibilidad 85,7 % y 100 %. Como son pocos casos (ver el intervalo de la sección 3), la métrica principal que reportamos es la precisión (66,7 % en prueba) junto con la sensibilidad. También comparamos con la regla que premia el equipo de producto ("SÍ si clics esperados ≥ 15 %"): da 28,6 % de exactitud en entrenamiento y 33,3 % en prueba, peor que la línea base.

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | 20 casos inventados de la ficha (14 de entrenamiento y 6 de prueba); pocos y no son personas reales. |
| ¿Qué métrica? | Precisión y sensibilidad; mostramos la exactitud pero siempre junto a la línea base. |
| ¿Comparado contra qué? | Contra responder siempre "NO" y contra la regla de clics. |
| ¿Quién lo midió? | Nosotras, con la división de entrenamiento y prueba fijada por la cátedra; no hubo evaluación externa. |

## 6. Riesgo: el camino completo

Datos → modelo → decisión → consecuencia. **Datos:** 20 casos inventados, pocos y posiblemente no representativos de los usuarios y del catálogo reales; además la afinidad se toma como dada y no sabemos cómo se calculó. **Modelo:** una neurona con una sola variable, que ignora la calificación y la duración: puede fallar con títulos de afinidad media pero muy buenos o muy largos. **Decisión:** SÍ si la afinidad es 0,58 o más; el umbral fija quién absorbe cada error. **Consecuencia:** lo que se pone en portada es lo único que el usuario mira, así que genera los datos del ciclo siguiente (bucle de retroalimentación): lo que el sistema no muestra nunca produce datos sobre sí mismo. Riesgo concreto: si el sistema solo recomienda lo que ya se parece a lo que el usuario mira (afinidad alta), la portada se vuelve repetitiva y nunca descubre gustos nuevos.

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | Armar el sistema para la ficha M04 a partir de la ficha y las bases | Neurona con variables que se conocen antes de decidir; parámetros buscados con los casos de entrenamiento | Usar "clics esperados" como entrada: es el indicador que premia el equipo de producto y no el objetivo |
| 2 | Calcular las matrices de confusión y las métricas | Matrices que cierran con 14 y 6 casos; exactitud, precisión, sensibilidad y especificidad | Presentar la exactitud como métrica principal: es un número engañoso con pocos casos |
| 3 | Corregir el sistema porque el profesor indicó que no puede dar 100 % en entrenamiento ni en prueba | Simplificar la neurona a una sola variable (afinidad) | Mantener tres y dos variables: separaban el entrenamiento al 100 % o con un solo tipo de error, señal de sobreajuste. También rechazamos agregar casos a los datos: la ficha pide usar solo sus 20 casos |
| 4 | Mejorar el README según los agujeros del evaluador | Explicar de dónde sale la afinidad, medir el umbral con una tabla y agregar el análisis de firmeza de las métricas | Ninguno |
| 5 | Redactar el README con la plantilla de la cátedra | La estructura de las 7 secciones | Ninguno por ahora |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | 8,5/10 (habilitadas para la mesa) | ¿De dónde sale la afinidad con el historial y en qué momento se calcula respecto de la decisión de poner el título en portada? Otras: si nunca se usaron clics, ¿qué cambió en el sistema para tener menos falsos positivos y se midió?; ¿qué pasa con las métricas de prueba si un solo caso cambia de lado? | Explicamos el origen de la afinidad y el supuesto de que se calcula antes de decidir; cambiamos el umbral de 0,50 a 0,58 y lo medimos en una tabla del cuaderno; agregamos el análisis de firmeza (un caso cambia de lado, intervalo de Wilson); completamos el link del Colab y esta bitácora |
