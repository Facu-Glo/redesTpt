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
**En el emisor**, los paquetes se generan periódicamente cada 20 milisegundos - ven la escalera perfecta. Esto tiene sentido porque así es como funciona el habla humana cuando se digitaliza.
Pero cuando estos paquetes viajan por la red y llegan al receptor la historia es muy diferente. Noten cómo los paquetes ya no llegan de forma uniforme. Algunos llegan rápido, otros lentos, algunos incluso llegan fuera de orden.

Ahora, el receptor tiene que tomar una decisión crítica: ¿cuánto tiempo espera antes de comenzar a reproducir los paquetes?

>
>Diapositiva 4


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

> ( PARTE 3 )

Este es el tradeoff fundamental: pérdida de paquetes versus retardo.

Y acá viene lo complicado: las condiciones de la red no son constantes.

Los autores observaron que los retardos pueden:

Fluctuar rápidamente - en segundos, no minutos
Tener picos súbitos - lo que llamaron 'spikes' - donde el retardo se multiplica por 5 o más instantáneamente
Variar según la hora del día, la ruta, la congestión

Entonces, no podemos usar un retardo fijo. Necesitamos algo que se adapte dinámicamente a estos cambios.
Y eso nos lleva a la solución...

# 3 Solución PLAYOUT ADAPTATIVO

* Diapositiva 3 con figura 2

La idea central es simple: en lugar de usar un retardo fijo durante toda la llamada, el receptor ajusta dinámicamente cuánto tiempo espera antes de reproducir los paquetes, adaptándose a las condiciones cambiantes de la red.

Primero, definamos las variables importantes:

> **Explicación de cada variable**

La magia está en cómo estimamos cuánto debe valer dᵢ para cada paquete."

El método funciona así:
Para el PRIMER paquete de cada talkspurt (segmento de habla):

 -> Ecuación 1

Para los SIGUIENTES paquetes del mismo talkspurt

 -> Ecuación 2

Esto significa que el retardo solo se ajusta al inicio de cada talkspurt, y dentro del talkspurt se mantiene la periodicidad original."

---

El paper evalúa cuatro algoritmos diferentes. La diferencia entre ellos está en cómo calculan d̂ᵢ (el retardo estimado).

- Algoritmo 1

α = 0.998 → muy conservador, cambios lentos
Es el algoritmo estándar usado en TCP

- Algoritmo 2 - Adaptación Asimétrica
Idea: reaccionar rápido a aumentos, lento a disminuciones

- Algoritmo 3 - Mínimo del Talkspurt Anterior
Extremadamente simple
Toma el retardo mínimo observado
Muy eficiente computacionalmente

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
