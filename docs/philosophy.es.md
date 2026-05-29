# Filosofía

Este repositorio es opinado. Leer las reglas, prompts y workflows sin
entender por qué existen te llevará a rechazar precisamente los que
desafían tus hábitos actuales — y son a menudo los que más vale la pena
mantener.

Este documento explica las creencias estructurales detrás de todo lo
que hay en `ai-first-dev-setup`. La versión en inglés está en
[`philosophy.md`](./philosophy.md).

## La apuesta

El desarrollo asistido por IA ya es el modo por defecto, no una novedad.
En el lapso de una semana laboral la mayoría de developers cae en uno
de dos resultados:

- **Lo adoptan de forma deliberada.** Configuran sus herramientas para
  que la IA tenga contexto preciso del proyecto, escriben prompts que
  producen output útil al primer intento, y tratan a la IA como un
  colaborador rápido con modos de fallo predecibles.
- **Lo usan por accidente.** Pegan código en un chat, aceptan lo que
  sale, y lo embarcan. Son ligeramente más rápidos que antes y
  sustancialmente menos fiables de lo que deberían ser.

Este repo es para el primer grupo. Los artefactos aquí son la
configuración, vocabulario y rituales que convierten a una IA de
"autocompletado con esteroides" a colaborador estructurado.

## Cinco creencias

### 1. El contexto vence al ingenio

El factor que más determina la calidad del output de una IA es cuánto
contexto preciso del proyecto tiene el modelo. Un prompt correcto y
estrecho contra un modelo que conoce tus convenciones supera a un
prompt brillantemente diseñado contra un modelo que no.

Por eso la mayor parte de este repo es configuración (`CLAUDE.md`,
`.cursorrules`, `agent.md`, configs de MCP), no plantillas de prompts.
Los prompts asumen que la configuración está en su sitio.

### 2. Planifica, confirma, ejecuta — siempre

Los LLMs rinden mejor con un plan claro y peor cuando tienen que
inferir intención desde peticiones vagas. El ritual de "planifica,
confirma, ejecuta" cuesta ~30 segundos y previene el modo de fallo más
caro del trabajo asistido por IA: cambios a medio implementar que
luego hay que deshacer.

Cada workflow y prompt de este repo refuerza esta cadencia. Si una
herramienta o modelo intenta saltársela, esa es una señal para frenar,
no para abrazar la velocidad.

### 3. La unidad mínima revisable

El código que se revisa en detalle captura bugs; el que se aprueba sin
mirar, no. Tanto humanos como IAs aprueban sin mirar cuando el diff es
grande. La solución es estructural — producir trabajo en cortes
pequeños que *exigen* una revisión real en cada paso.

Verás este principio empotrado en prompts ("muéstrame el diff tras
cada corte"), workflows ("pausa entre tandas") y hasta en las
`.cursorrules` ("output progresivo"). No es opcional.

### 4. El spec es el artefacto

Un error frecuente en el trabajo asistido por IA es iterar sobre el
*código* cuando la confusión subyacente está en el *spec*. El código
parece mal porque el spec estaba mal; reescribir el código sin
reescribir el spec sólo desplaza el bug.

Apostamos por trabajo spec-first: escribe el spec más pequeño que
capture el cambio, somételo a crítica con la IA, y luego genera código
a partir de él. Cuando la realidad de la implementación fuerza un
cambio de spec, cambia el spec primero como commit separado. El spec es
la fuente de verdad; el código es su proyección.

### 5. Verifica el comportamiento, no el output

El trabajo de la IA termina cuando el comportamiento coincide con el
requisito, no cuando dice "Listo". Que los tests pasen es necesario
pero no suficiente. Para cambios de cara al usuario: pulsa el botón tú
mismo. Para cambios de backend: golpea el endpoint. Para cambios de
datos: ejecuta una query real contra datos reales. El coste de esta
verificación es bajo; el de saltársela lo pagan los usuarios.

## Lo que excluimos deliberadamente

- **Prompts para "mejorar mi código".** La intención vaga produce
  output vago. Cada prompt aquí tiene una condición de uso específica.
- **"Personas IA"** ("Eres un ingeniero 10x llamado Bob"). Teatro, no
  señal. Los modelos frontera funcionan mejor con instrucciones
  técnicas directas que con priming de persona.
- **Asunciones de modelo único.** Los prompts se prueban contra
  múltiples modelos actuales. Si sólo funcionan en uno, son demasiado
  frágiles para distribuirse.
- **Herramientas más allá del asistente + IDE.** Sin CLIs propios, sin
  frameworks de orquestación. La superficie de herramientas es la que
  ya tienes; el apalancamiento está en cómo las usas.

## Cómo discrepar con este repo

Si una regla, prompt o workflow aquí no encaja con tu situación:
cámbialo. Hazle fork. Abre un PR con el cambio y el motivo. Este repo
es opinado pero no dogmático — la audiencia es de profesionales
tomando decisiones específicas a su contexto, no novatos siguiendo un
guión.

La única petición: cuando discrepes, cambia el artefacto por escrito
en lugar de cambiarlo sólo en la práctica. La divergencia entre lo que
dice tu config y lo que realmente haces es peor que cualquiera de los
extremos.
