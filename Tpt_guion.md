# 1 introducción

Diapositiva 1 -> Presentación del grupo y tema a tratar, dar contexto histórico del paper
## Presentación del grupo
[Presentacion] Hola, soy López Facundo Gabriel y junto a mi compañero Rivero Julian vamos a presentar el siguiente tema: 
Mecanismos Adaptativos de Reproducción para Aplicaciones de Audio Empaquetado en Redes de Área Amplia

> **Explicacion sobre el titulo del paper** 

[Contexto del Paper] Es un paper, publicado a finales de los años 90 (1994), escrito por Ramachandran Ramjee, Jim Kurose y Don Towsley, de la Universidad de Massachusetts.
En esa época, el interés por las aplicaciones de audio y voz sobre IP estaba explotando, y se buscaba demostrar que el Internet de área amplia podía soportar audio interactivo de calidad.

Los principios que desarrollaron estos investigadores son la base tecnológica de todas las aplicaciones de voz y video que usamos diariamente:

Cada llamada de Zoom o Google Meet
Cada llamada de voz por WhatsApp, Telegram o Signal
Cada stream en vivo en Twitch o YouTube

Todos estos servicios usan variantes de los mecanismos adaptativos que vamos a ver a continuación.

# 2 Problema a resolver

¿Puede Internet, con sus retardos variables, soportar audio interactivo en tiempo real?
El problema central era este: los paquetes de audio viajan por la red experimentando retardos diferentes e impredecibles - lo que llamamos jitter. El receptor necesita reproducir estos paquetes de forma periódica para que el audio suene bien, pero ¿cuánto tiempo debe esperar antes de reproducir cada paquete?

* Figura 1 en la presentación.

Veamos gráficamente el problema.

El gráfico del **emisor** (_sender_) muestra un patrón de escalera regular, con escalones de igual tamaño espaciados uniformemente:

- **¿Qué significa?** Significa que el audio se está muestreando y **los paquetes se generan periódicamente**.
- **Idealidad:** Esta es la naturaleza idealizada de la fuente de audio: cuando alguien está hablando (_talkspurt_), se produce un paquete cada intervalo de tiempo fijo (por ejemplo, cada 20 milisegundos). La pendiente constante de la línea de escalera indica una **tasa de generación de paquetes constante**.

Pero cuando estos paquetes viajan por la red y llegan al receptor la historia es muy diferente.
Los paquetes no llegan al receptor a intervalos de tiempo constantes. Esto es causado por el **retraso variable de la red** (_delay jitter_), es decir, el tiempo que tarda cada paquete en atravesar la red es aleatorio y diferente para cada paquete

Ahora, el receptor tiene que tomar una decisión crítica: ¿cuánto tiempo espera antes de comenzar a reproducir los paquetes?

Fíjense en estos dos escenarios:

Escenario 1 - Si el receptor comienza a reproducir en el tiempo t₁:

El retardo es corto
PERO los paquetes 6, 7 y 8 todavía no llegaron 
Esos paquetes se pierden - llegaron demasiado tarde
El usuario escucha cortes en el audio

Escenario 2 - Si espera hasta t₂:

Todos los paquetes llegaron a tiempo
PERO hay un retardo mucho mayor
La conversación se siente no natural, como hablar por walkie-talkie

Este es el tradeoff fundamental: pérdida de paquetes versus retardo.
Y acá viene lo complicado: las condiciones de la red no son constantes.

Entonces, no podemos usar un retardo fijo. Necesitamos algo que se adapte dinámicamente a estos cambios.
Y eso nos lleva a la solución...

>Diapositiva 3

Pero antes, ahora que entendemos el problema del _jitter_, necesitamos una manera de medir y gestionar los retrasos. Para esto, el _paper_ define las variables clave utilizando la Figura 2.

Es un diagrama de línea de tiempo para un solo paquete, el **paquete i**. Muestra el ciclo de vida de este paquete desde que se crea hasta que se reproduce, y define cómo se relaciona el retraso de la red con nuestra solución.

##### **Puntos Clave en el Tiempo**

Estos son los tres momentos críticos en la vida de nuestro paquete:

-  $t_i$​ (Tiempo de Generación): Es el momento en que el paquete i es creado en el **emisor**. Este es nuestro punto de partida.
    
- $a_i$​ (Tiempo de Llegada): Es el momento en que el paquete i llega al receptor La diferencia entre ai​ y ti​ es el retraso total de la red.
    
- $p_i$​ (Tiempo de Playout): Es el momento en que el paquete i es reproducido y escuchado. Este es el instante que nuestro algoritmo tiene que decidir.

**Intervalos de Tiempo (Los Retrasos)**
La figura desglosa el retraso total en dos componentes principales que actúan como la clave de nuestro _tradeoff_.

1. **$n_i$​ (Retraso de Red - _Network Delay_):**
    
    - Es el tiempo que el paquete pasó viajando: **ai​−ti​**.
        
    - Incluye el retraso de propagación constante (Dprop​) y, crucialmente, el **retraso variable en cola (vi​)**, que es el _jitter_ que queremos mitigar.
        
2. **$b_i$​ (Tiempo en Búfer - _Buffer Time_):**
    
    - Este es el tiempo que el paquete pasa **esperando en el receptor**: **pi​−ai​**.
        
    - Este es el **mecanismo de compensación**. Lo usamos para absorber la variabilidad de ni​. Un bi​ más grande significa que esperamos más, pero tenemos más margen para el _jitter_.
# 3 Solución PLAYOUT ADAPTATIVO
> Diapositiva 5


Entonces la idea central es simple: en lugar de usar un retardo fijo durante toda la llamada, el receptor ajusta dinámicamente cuánto tiempo espera antes de reproducir los paquetes, adaptándose a las condiciones cambiantes de la red.

La magia está en cómo estimamos cuánto debe valer dᵢ para cada paquete."

El método funciona así:
Para el PRIMER paquete de cada talkspurt (segmento de habla):

 -> Ecuación 1

Para los SIGUIENTES paquetes del mismo talkspurt

 -> Ecuación 2
 
Que son $\hat d_i$ y $\hat v_i$?
$\hat d_i$ Son estimaciones del retardo promedio de la red, osea cuanto suelen tardar los paquetes en llegar.
$\hat v_i$ Son las estimaciones de la variación del retardo, osea que tanto cambia ese tiempo de llegada.

---

El paper evalúa cuatro algoritmos diferentes. La diferencia entre ellos está en cómo calculan $\hat d_i$ (el retardo estimado).

- Algoritmo 1

α = 0.998
un valor **muy alto**, lo que significa que el historial pesa muchísimo
Es muy **suave**, muy **estable**, pero también muy **lento** para reaccionar a cambios

- Algoritmo 2 - Adaptación Asimétrica

 Intenta ser más listo: usa **dos pesos diferentes**
 Cuando el retardo **sube**: α = 0.75 (reacciona rápido)
 Cuando **baja**: β = 0.998002 (reacciona lento)
 La idea venía de TCP - detectar congestión rápido, recuperarse despacio

- Algoritmo 3 - Mínimo del Talkspurt Anterior

Extremadamente simple
Toma el retardo mínimo observado
Asume que el mínimo es una buena 'línea base'
Problema: **ignora completamente la variabilidad**

- Algoritmo 4 - Detección de Spikes
Este es la principal contribución del paper.
Detecta y se adapta específicamente a los 'spikes' de retardo que observaron en Internet real.

# Algoritmo 4 Detección de Spikes

Al analizar trazas reales de tráfico de audio entre distintos sitios de Internet, los autores descubrieron un patrón recurrente que llamaron 'spikes' de retardo.

* Diapositiva -> figura 5

Muestra el retardo de red en función del tiempo de llegada de los paquetes.

Spike:
Aumento súbito: El retardo salta de ~0.2 segundos a ~1.2 segundos - un aumento de 1 segundo en un instante
Ráfaga de llegadas: Inmediatamente después, unos 50 paquetes llegan casi simultáneamente en apenas 200 milisegundos.
En condiciones normales esperaríamos 1 paquete cada 20ms, o sea 10 paquetes en 200ms
Retorno gradual: El retardo vuelve progresivamente a valores normales
Este fenómeno se debe típicamente a congestión súbita en algún router del camino, seguida de un vaciado rápido de su buffer.

Los tres primeros algoritmos no se adaptan bien a estos spikes:
- Algoritmo 1
Con α = 0.998, es demasiado lento para reaccionar
Tarda muchos paquetes en subir su estimación
Resultado: muchas pérdidas durante el spike

- Algoritmo 2 (Mills):
Aunque sube más rápido (α=0.75), baja muy lento (β=0.998)
Una vez terminado el spike, mantiene el retardo alto innecesariamente
Resultado: retardo promedio elevado

- Algoritmo 3 (Mínimo):
Solo usa el mínimo del talkspurt anterior
Si hay un spike al inicio del nuevo talkspurt, no tiene información para adaptarse
Resultado: pérdidas impredecibles

---
Algoritmo 4 -> explicar que tiene 2 modos: Normal y Impulse
-> como se detecta el spike
-> como se detecta el fin del spike
