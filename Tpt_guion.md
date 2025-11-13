# 1 introducción

Diapositiva 1 -> Presentación del grupo y tema a tratar, dar contexto histórico del paper
## Presentación del grupo
[Presentacion] Hola, soy López Facundo Gabriel y junto a mi compañero Rivero Julian vamos a presentar ~~el  siguiente tema:~~ 
Mecanismos Adaptativos de Reproducción para Aplicaciones de Audio Empaquetado en Redes de Área Amplia

En esta presentación vamos a ver diferentes algoritmos de reproducción de paquetes que se utilizan en aplicaciones de audios en tiempo real.  El objetivo de estos algoritmos va a ser ajustar dinámicamente el tiempo de reproducción del audio de tal forma que una conexión inestable afecte lo menos posible la experiencia final del usuario.

[Contexto del Paper] ~~Es~~ un paper, publicado a finales de los años 90 (1994), escrito por Ramachandran Ramjee, Jim Kurose y Don Towsley, de la Universidad de Massachusetts.
En esa época, el interés por las aplicaciones de audio y voz sobre IP estaba explotando, y se buscaba demostrar que el Internet de área amplia podía soportar audio interactivo de calidad.

~~Los principios que desarrollaron estos investigadores son la base tecnológica de todas las aplicaciones de voz y video que usamos diariamente:~~

~~Cada llamada de Zoom o Google Meet
Cada llamada de voz por WhatsApp, Telegram o Signal
Cada stream en vivo en Twitch o YouTube~~

~~Todos estos servicios usan variantes de los mecanismos adaptativos que vamos a ver a continuación.~~

# 2 Problema a resolver

~~¿Puede Internet, con sus retardos variables, soportar audio interactivo en tiempo real?~~
El problema central era este: los paquetes de audio viajan por la red experimentando retardos diferentes e impredecibles - lo que llamamos jitter. El receptor necesita reproducir estos paquetes de forma periódica para que el audio suene bien, pero ¿cuánto tiempo debe esperar antes de reproducir cada paquete?

>  Diapositiva 2 Figura 1.

Veamos gráficamente el problema.

El gráfico del **emisor** (_sender_) muestra un patrón de escalera regular, con escalones de igual tamaño espaciados uniformemente:

- **¿Qué significa?** Significa que ~~el audio se está muestreando y~~ **los paquetes se generan periódicamente**.

Pero cuando estos paquetes viajan por la red y llegan al receptor la historia es muy diferente.
Los paquetes no llegan al receptor a intervalos de tiempo constantes. Esto es causado por el **retraso variable de la red** (_delay jitter_), es decir, el tiempo que tarda cada paquete en atravesar la red es aleatorio y diferente para cada paquete

~~Ahora, el receptor tiene que tomar una decisión crítica: ¿cuánto tiempo espera antes de comenzar a reproducir los paquetes?~~

Fíjense en estos dos escenarios:

Escenario 1 - Si el receptor comienza a reproducir en el tiempo $t_1$:

El retardo es corto
PERO los paquetes 6, 7 y 8 todavía no llegaron 
Esos paquetes se pierden - llegaron demasiado tarde
El usuario escucha cortes en el audio

Escenario 2 - Si espera hasta $t_2$:

Todos los paquetes llegaron a tiempo
PERO hay un retardo mucho mayor
La conversación no se siente natural.

~~Entonces, no podemos usar un retardo fijo. Necesitamos algo que se adapte dinámicamente a estos cambios.
Y eso nos lleva a la solución...~~

>Diapositiva 3

~~Pero antes, ahora que entendemos el problema del _delay jitter_, necesitamos una manera de medir y gestionar los retrasos. Para esto, el _paper_ define las siguientes variables.~~

~~Es un diagrama de línea de tiempo para un solo paquete, el **paquete i**. Muestra el ciclo de vida de este paquete desde que se crea hasta que se reproduce,y define cómo se relaciona el retraso de la red con nuestra solución.~~


# 3 Solución PLAYOUT ADAPTATIVO

> Diapositiva 4

Antes de presentar los algoritmos veamos como se calculan los tiempos de reproducción.

El método funciona así:
Para el PRIMER paquete de cada segmento de habla (talkspurt):

El momento en que el paquete i es reproducido es igual a el momento en que el paquete i es emitido mas la estimacion del retardo promedio de la red  mas 4 por la estimacion de la variacion del retardo.

 -> Ecuación 1

Una vez que el tiempo de _playout_ (reproducción) inicial ha sido decidido por la fórmula anterior, **el objetivo es reproducir el resto de los paquetes al mismo ritmo constante** que tenían cuando fueron generados en el emisor. Esto garantiza que la voz suene natural y fluida, sin las pausas o aceleraciones causadas por el _jitter_ de la red

 -> Ecuación 2
 
Que son $\hat d_i$ y $\hat v_i$?

$\hat d_i$ Son estimaciones del retardo promedio de la red, osea es el **tiempo de viaje promedio estimado** de los paquetes de audio a través de la red.

$\hat v_i$ Son las estimaciones de la variación del retardo, osea Es una estimación de **cuánto varía** el retardo real de los paquetes con respecto al promedio

---

El paper evalúa cuatro algoritmos diferentes. La diferencia entre ellos está en cómo calculan $\hat d_i$ (el retardo promedio estimado).

---
- ==Algoritmo 1==

> Diapositiva 6

El valor α=0.998 se elige deliberadamente para que el algoritmo sea **conservador**:

- **Inercia Alta:** Un α tan cercano a 1 significa que la estimación promedio actual ($\hat d_i$​) depende gran parte de la estimación anterior ($\hat d_{i−1}$​).
- **Lenta Reacción:** El nuevo paquete que llega ($n_i$​) no influye mucho en este calculo. Esto hace que el algoritmo sea  **lento para adaptarse** si las condiciones de la red cambian abruptamente (por ejemplo, si el retraso promedio aumenta).
---
- ==Algoritmo 2 - Adaptación Asimétrica==

> Diapositiva 7

El problema del Algoritmo 1 (α=0.998) era que, cuando llegaba un paquete con un retraso muy alto ($n_i$​), la estimación $\hat d_i$​ era muy lenta para ajustarse. Esto causaba la pérdida de ese paquete y los siguientes.
El Algoritmo 2 trata de  resolver esto utilizando **dos valores diferentes para $\alpha$** al actualizar la estimación del retardo promedio $\hat d_i$​:

 Cuando el retardo **sube**: α = 0.75 (reacciona rápido)
 Cuando **baja**: β = 0.998002 (reacciona lento)

~~- **Ventaja:** Resuelve el problema de la **pérdida de paquetes** causada por picos. El sistema se adapta casi instantáneamente a un empeoramiento de la red, elevando rápidamente el tiempo de _playout_ programado ( $p_i$​ ).~~
    
- ~~**Desventaja (Hipo):** Aunque resuelve el problema de la pérdida, el $\hat d_i​$​ **no baja lo suficientemente rápido** una vez que el pico de retraso desaparece. Esto significa que, después de un pico, el usuario está forzado a experimentar un **retraso total innecesariamente alto** durante un largo periodo de tiempo ~~hasta que la estimación $\hat d_i​$ finalmente decae lentamente.~~

---

- ==Algoritmo 3 - Mínimo del Talkspurt Anterior==

> Diapositiva 8

Extremadamente simple
Toma el retardo mínimo observado
Asume que el mínimo es una buena 'línea base'
~~Problema: **ignora completamente la variabilidad**~~

---
- Algoritmo 4 - Detección de Spikes
Este es la principal contribución del paper.
Detecta y se adapta específicamente a los 'spikes' de retardo que observaron en Internet real.

# Spikes

Todos estos algoritmos iniciales no tienen en cuenta los llamados **picos de retardo**, momentos en los que los paquetes sufren un aumento brusco de demora y luego llegan casi simultáneamente, con pocos milisegundos de diferencia.

El **Algoritmo 1**, muy conservador, tarda demasiado en reaccionar cuando cambia la congestión de la red, **descartando muchos paquetes** por llegar tarde.

El **Algoritmo 2** detecta rápidamente el aumento de retardo, pero luego **mantiene un retardo innecesariamente alto**, demorando en volver a la normalidad.

El **Algoritmo 3** carece de información suficiente para detectar el estado real de la red, por lo que **no ajusta su tolerancia** y también termina descartando numerosos paquetes.

Así es como **surge el Algoritmo 4**, diseñado específicamente para **reconocer y adaptarse a estos picos**, logrando una mejor calidad y menor pérdida de paquetes.

# Algoritmo 4

El algoritmo 4, tiene 2 modos:

El modo normal que es identico al algoritmo 1, pero menos conservador, con un alpha de 0.875

El modo impulso, que entra luego de encontrarse con una diferencia de retardos superior a un umbral llamado spike_threshold, en el que el algoritmo sigue la curva del pico de retardo, ajustándose rápidamente a los cambios bruscos y volviendo al modo normal una vez que la red se estabiliza.

# Comparacion

Luego de analizar las pruebas que se encuentran en el paper, se puede observar que el algoritmo 4, que incorpora detección y seguimiento de picos de retardo, ofrece el mejor desempeño. Este enfoque logra **menor pérdida de paquetes** y **menor retardo promedio de reproducción** en comparación con los otros métodos.  

# Conclusion

Una adaptación rápida frente a variaciones bruscas de la red influye significativamente en la **calidad percibida del audio** por el usuario y la **interactividad** de la aplicacion. Encontrar el balance entre estos dos, es una tarea fundamental de la que se encargan los algoritmos vistos. 

En particular, el algoritmo 4 demostró ofrecer los **mejores resultados**, al adaptarse eficazmente a los picos de retardo y reducir la pérdida de paquetes.