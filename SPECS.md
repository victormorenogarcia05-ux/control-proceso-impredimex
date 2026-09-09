# SPECS.md — Control de Procesos (Calidad)

## Especificaciones funcionales del sistema

Este documento es la **fuente de verdad** del comportamiento de la aplicación.
Cualquier cambio futuro debe partir de actualizar primero estas specs y luego
implementar el código.

**Versión objetivo:** 1.0 — reconstrucción desde cero
**Fecha:** 9 de septiembre de 2026
**Metodología:** Spec-Driven Development (SDD)

> **Por qué se reconstruye.** La versión anterior se usó de verdad entre abril y
> julio de 2026: 1 772 inspecciones en ocho máquinas, en los tres turnos. Se
> abandonó en julio y Calidad volvió a inspeccionar en papel. El motivo no fue
> que estorbara, sino que **resolvía una sola forma de inspeccionar** cuando el
> proceso real tiene varias etapas y dos enfoques distintos.
>
> El respaldo de esa base está guardado fuera del repositorio. La aplicación
> nueva **arranca limpia**: no se migran datos.

---

## Lo que enseñó la versión anterior

Estos hallazgos vienen de analizar el respaldo, y cada uno se traduce en una
regla de esta versión.

**Las acciones correctivas nunca se cerraban.** Las 304 quedaron en estado
`abierta`, sin responsable, sin causa y sin fecha de cierre. La herramienta
señalaba problemas y no ofrecía manera de resolverlos. Es lo que más pesó en el
abandono.

**Nadie sabía quién inspeccionaba.** 1 737 de las 1 772 inspecciones tienen como
inspector la palabra genérica «Calidad», porque las tres contraseñas eran
compartidas y el perfil se autoasignaba ese nombre. Tres meses de evidencia sin
responsable identificable.

**No se registraba la causa.** La aplicación anotaba *qué* falló pero nunca *por
qué*. Por eso hoy es imposible calcular qué variable de proceso provoca cada
defecto, que es justamente lo que esta versión necesita.

**El cumplimiento promedio fue 99.1%.** Con un número así, el indicador deja de
informar. Cuando marcar todo conforme es más rápido que registrar un hallazgo,
el dato se degrada solo.

**La taxonomía de defectos era demasiado gruesa.** La aplicación registraba
«Defectos de impresión» y «Tonos». El histórico de merma de la empresa distingue
**40 defectos específicos**: raya, lagrimeo, manchas, velo, remosqueo, repinte,
impresión faltante, desprendimiento. La diferencia es decisiva: «defectos de
impresión» no apunta a ninguna variable de proceso, mientras que «raya» apunta a
rasqueta, cilindro y sustrato. Con la categoría gruesa, la matriz de la SPEC-005
habría sido imposible de determinar.

**Los defectos se concentran por máquina.** Analizando 1 095 registros de merma
de 2025 y 2026: cuatro líneas de 23 explican el 80% de los metros rechazados
—RT7 con 36.6%, RT6 con 20.1%, FL1 con 19.2% y FL4 con 6.8%—. Por cliente hacen
falta 12 de 52 para llegar al 80%, y por etiqueta 115 de 336. El foco de la
inspección va por máquina.

---

# SPEC-001 — Acceso a la aplicación

**Estado:** pendiente

### Precondiciones
- La persona existe en `colaboradores` del proyecto **impredimex-suite**, con
  `estatus` en `ACTIVO`
- Su campo `apps` incluye `calidad`
- Tiene cuenta en Firebase Auth del proyecto suite

### Flujo principal
1. Sistema muestra la pantalla de acceso con nómina y clave
2. Sistema arma `<nómina>@impredimex.local` y llama a
   `signInWithEmailAndPassword` contra Auth de la suite
3. Sistema lee `colaboradores/<nómina>`, valida `estatus` y `apps`
4. Sistema toma el papel de `roles.calidad`
5. Sistema abre además una sesión anónima en el proyecto propio

### Reglas de negocio
- **Desaparecen las tres contraseñas compartidas** de la versión anterior:
  `IMPREDIMEX`, `imagenes` y `Calidad`. Las dos primeras eran además las mismas
  de otras aplicaciones, así que quien conocía las de EPP entraba aquí.
- **El inspector es quien inició sesión.** No se elige de una lista ni se
  autoasigna un nombre genérico. Es la corrección más importante respecto de la
  versión anterior.
- **Ausencia de papel equivale al más bajo.**

---

# SPEC-002 — Papeles

**Estado:** pendiente

| Papel | Quién | Qué hace |
|---|---|---|
| `INSPECTOR` | Inspectores de calidad | Inspecciona en cualquier etapa, registra hallazgos y causas |
| `SUPERVISOR` | Supervisores de producción | Recibe avisos de paro, atiende y cierra acciones correctivas |
| `ADMIN` | Jefatura de calidad | Todo lo anterior, más catálogos, matriz de causas y criterios de muestreo |

### Reglas de negocio
- **Inspeccionan todos los inspectores de calidad**, cada uno con su cuenta.
- **Quien detecta no cierra.** El inspector levanta el hallazgo; el supervisor lo
  atiende. Que la misma persona haga ambas cosas anula el control.

---

# SPEC-003 — Etapas de inspección

**Estado:** pendiente

Dos etapas, no una:

**Arranque.** Antes de liberar la corrida. Se verifica que las condiciones para
producir bien estén dadas.

**En proceso.** Durante la corrida, en los recorridos del inspector.

### Reglas de negocio
- **Se descartó la etapa de fin de corrida.** Existía en la idea original por la
  rutina de recorridos, pero lo que se revisa ahí ya se cubrió durante la
  corrida. Agregarla solo produce ruido.
- La etapa determina qué se pregunta: el arranque mira condiciones previas; la
  inspección en proceso mira producto y variables.

---

# SPEC-004 — Los dos enfoques

**Estado:** pendiente

Esta es la idea central de la reconstrucción y lo que la versión anterior no
distinguía.

### Enfoque 1 — Producto
Se inspecciona el semiterminado o el terminado. El resultado es **binario: pasa
o no pasa**.

Su límite es que **cuando detecta, el daño ya ocurrió**. Si el semiterminado no
pasa, hay pérdida y poco que hacer.

### Enfoque 2 — Variables de proceso
Se inspeccionan las condiciones que, de desviarse, provocarían el defecto. Es
preventivo: se actúa antes de que haya producto malo.

### Cómo se conectan
El inspector no debe adivinar qué variables vigilar. La aplicación se lo dice:

1. Conoce los **defectos históricos de esa máquina**, ordenados por frecuencia
2. Conoce, por la matriz de la SPEC-005, **qué variables provocan cada defecto y
   en qué proporción**
3. Propone vigilar las variables que explican la mayor parte de los defectos
   probables de esa máquina

### El foco se calcula en dos niveles

**Primer nivel, por línea.** Es donde se concentra la merma y el perfil de cada
equipo es claramente distinto:

| Línea | Sus principales defectos |
|---|---|
| RT7 | Lagrimeo 19.6%, raya 17.6%, impresión faltante 13.5%, registro 11.8% |
| RT6 | **Raya 42.9%**, lagrimeo 13.6%, manchas 11.3%, registro 10.9% |
| FL1 | Manchas 18.4%, raya 16.4%, repinte 9.3%, remosqueo 7.6% |
| FL4 | **Tonos 17.6%**, impresión faltante 14.1%, manchas 11.7% |

**Segundo nivel, por etiqueta.** Dentro de una misma línea, etiquetas distintas
fallan por razones distintas. En la RT7, sus tres etiquetas más costosas: la
28691 es 100% impresión faltante, la 29355 es desprendimiento y lagrimeo, y la
28947 es velo. Si la etiqueta que se está corriendo tiene historia suficiente,
el foco se afina con ella; si no, se queda en el de la línea.

### Reglas de negocio
- **El foco es por máquina, no general.** El cliente no sirve para dirigir la
  inspección: la merma se reparte entre demasiados.
- **La inspección registra qué etiqueta se está corriendo.** La versión anterior
  no lo capturaba, y sin ese dato el segundo nivel del foco es imposible.
- **La prioridad se mide por metros rechazados, no por número de casos.** Es lo
  que hace el histórico de merma y es más honesto: 16 casos de tonos cuestan más
  que 27 de punteado. Contar hallazgos trata igual una raya de 50 metros que una
  de 5 000.
- **La propuesta es una guía, no una restricción.** El inspector puede revisar
  cualquier variable; la aplicación resalta las que más rinden.
- **Hace falta un mínimo de historia para afinar por etiqueta.** Con dos o tres
  registros, el porcentaje es ruido. El umbral lo define Calidad.

### Dos ventanas de tiempo

**Ventana de foco: tres meses naturales.** Es la que determina el peso de cada
defecto y lo que se le propone vigilar al inspector.

Al abrir la aplicación se posiciona sola en **el mes en curso y los dos
anteriores**: abierta en septiembre muestra septiembre, agosto y julio. No es una
ventana móvil de noventa días, son meses completos, que es como se lee un
tablero de calidad.

**El inspector puede mover la ventana.** Si quiere ver otro trimestre, lo
selecciona. La aplicación muestra siempre qué periodo está usando, para que nadie
interprete un número creyendo que es del mes actual.

> **El mes en curso está incompleto.** Abierta el día 9, ese mes lleva nueve días
> de datos y pesa menos que los dos completos. La aplicación lo indica en vez de
> presentarlo como si fuera un mes cerrado.

**Ventana de comportamiento: todo el histórico.** Muestra la evolución de cada
defecto en el tiempo, para saber si **disminuyó, se mantuvo o aumentó**. No pesa
en el foco, pero da el contexto que el foco por sí solo no da.

Que el peso venga de los tres meses no es un detalle: comparando ambas ventanas
sobre los 1 095 registros de 2025 y 2026, el orden cambia de verdad.

| Todo el histórico | Últimos tres meses |
|---|---|
| Raya 19.3% | Raya **25.6%** |
| Lagrimeo 10.9% | Manchas **14.2%** |
| Manchas 10.3% | Lagrimeo 13.0% |
| Registro 8.1% | **Falta de presión 7.5%** |
| Impresión faltante 6.9% | Registro 7.3% |
| Adhesivo 4.9% | Fuera de registro 5.4% |

La falta de presión aparece de la nada en la ventana corta; impresión faltante y
adhesivo salen del podio. Y por línea el contraste es mayor: en la **FL4** el
principal defecto histórico son los tonos con 18%, pero en los últimos tres meses
es **falta de presión con 28%**. Un inspector guiado solo por el histórico
vigilaría lo que ya se corrigió.

- **El foco pesa por la ventana de tres meses, sin descartar el comportamiento
  histórico.** La segunda ventana no se usa para ordenar, se usa para entender.
- **En una línea de poca actividad, la ventana se amplía sola.** Si en los tres
  meses apenas hay registros de esa máquina, el foco se calcularía sobre ruido.
  La aplicación retrocede hasta juntar historia suficiente y **dice con qué
  periodo lo hizo**, para que el inspector sepa que está viendo algo más viejo. La
  alternativa —mostrar un foco débil con la misma seguridad que uno sólido— es
  peor que no mostrarlo.
- **La tendencia se muestra junto a cada defecto del foco.** Saber que la raya
  pasó de 19.3% a 25.6% —que va en aumento— cambia la urgencia con que se
  atiende, frente a un defecto que baja y probablemente ya tiene una acción
  surtiendo efecto.

---

# SPEC-005 — Matriz defecto ↔ variable

**Estado:** pendiente — **el contenido lo define una junta de calidad,
producción e ingeniería de procesos**

### Qué es
Para cada defecto, qué variables de proceso lo provocan y con qué peso relativo.

### Reglas de negocio
- **Es un catálogo editable dentro de la aplicación, no código.** La junta captura
  el resultado sin depender de una nueva versión, y lo corrige cuando cambie un
  material o se rectifique un cilindro.
- **Puede diferir por máquina.** Lo que provoca variación de tono en rotograbado
  no tiene por qué ser lo mismo que en flexografía.
- **Se siembra con el criterio de la junta y se corrige con los datos.** Al
  principio los pesos son opinión experta; conforme se acumulen causas
  registradas (SPEC-007), la aplicación muestra el porcentaje real medido junto
  al estimado, para que la junta ajuste.
- **La aplicación funciona con la matriz vacía.** En ese caso solo muestra los
  defectos históricos, sin sugerir variables. No es un requisito para publicar.

### Pendiente de definir
El contenido. Con **13 de los 40 defectos se cubre el 80%** de los metros
rechazados, así que no hace falta agotar la lista para que la matriz sirva. Por
peso conviene empezar por: raya 19.3%, lagrimeo 10.9%, manchas 10.3%, registro
8.1%, impresión faltante 6.9%, adhesivo 4.9% y velo 4.8%.

Y conviene revisarlos por línea, porque la causa puede diferir: la raya es el 43%
de la merma de la RT6 y el 17.6% de la RT7.

---

# SPEC-006 — Muestreo por AQL

**Estado:** pendiente

### Qué hace
La aplicación **calcula el plan de muestreo**; el inspector no consulta tablas.

A partir del tamaño del lote y del criterio vigente, indica cuántas piezas
revisar, cuántos defectos se aceptan y a partir de cuántos se rechaza.

### Reglas de negocio
- **El criterio lo define Calidad**, no viene fijo: nivel de inspección, valor de
  AQL y severidad. Se configura en la aplicación.
- **Puede diferir por tipo de producto o cliente**, si Calidad así lo decide.
- **Las tablas del estándar viven en la aplicación**, y la decisión de aceptar o
  rechazar queda registrada junto con el plan que se aplicó. Sin eso, un
  resultado no es auditable meses después.

### Pendiente de definir
- Qué niveles de inspección y valores de AQL usa Calidad
- Si la aplicación debe aplicar las **reglas de cambio de severidad** del
  estándar —pasar a inspección rigurosa tras cierto número de lotes rechazados,
  y volver a normal— o si esa decisión la toma Calidad a mano. Automatizarlo es
  más fiel al estándar; hacerlo a mano es más simple y transparente.

---

# SPEC-007 — Hallazgo, paro y acción correctiva

**Estado:** pendiente

### Flujo principal
1. El inspector detecta una no conformidad y la registra
2. Sistema **exige la causa**: qué variable de proceso la provocó. Si el inspector
   no la sabe, puede marcarla como no determinada, y eso también es un dato
3. Sistema notifica al supervisor de producción
4. Si procede, el inspector solicita **paro de máquina** y queda registrado
5. El supervisor atiende, registra qué se corrigió y **cierra** la acción
6. Sistema conserva quién detectó, quién atendió, cuándo y con qué resultado

### Reglas de negocio
- **Una acción correctiva se puede cerrar.** Es la corrección más importante
  respecto de la versión anterior, donde las 304 quedaron abiertas para siempre.
- **La causa es obligatoria.** Sin ella, la matriz de la SPEC-005 nunca se
  corrige con datos reales y se repite el vacío de la versión anterior.
- **Quien detecta no cierra** (SPEC-002).
- **El paro de máquina se registra como tal**, con hora de solicitud. Hoy ese
  aviso ocurre de viva voz y no queda rastro.

---

# SPEC-008 — Catálogo de variables de calidad

**Estado:** pendiente

### El catálogo de defectos
Se adopta la **taxonomía de 40 defectos** del histórico de merma de la empresa,
no la de la versión anterior. Es la que usa producción para medir el costo, y es
lo bastante específica como para poder asociarle causas.

### Qué se hereda de las variables
Los **54 criterios** de la versión anterior, con su método de medición,
especificación y tolerancia. Están repartidos en genérica, máquinas, producto,
refilado, pegado, revisado y corte, más criterios específicos de la OMET X6 530
y la Huafeng HYA.

Es el único contenido irreemplazable de la versión anterior y se conserva.

### Reglas de negocio
- **El catálogo vive en la base, no en el código.** En la versión anterior estaba
  escrito en el archivo y por eso nunca se pudo ajustar sin tocar programación.
- **De los 54 criterios, en la práctica se usaron 21 por inspección.** Al cargar
  el catálogo conviene revisar cuáles siguen vigentes: las secciones de
  seguridad, materiales y documentación no reprobaron ni una vez en tres meses,
  lo que sugiere que se marcan por trámite.

---

# SPEC-011 — Historial de merma como semilla del foco

**Estado:** pendiente
**Origen:** el histórico de merma de la empresa, 1 095 registros de 2025 y 2026

### Por qué existe
Sin historia, la aplicación tarda meses en poder decirle algo con fundamento al
inspector, y ese fue justamente el vacío de la versión anterior. Cargando el
histórico, **el foco de la SPEC-004 funciona desde el primer día** con dos años
de datos reales.

### Flujo principal
1. `ADMIN` carga el archivo de merma en formato Excel
2. Sistema reconoce fecha, pedido, línea, etiqueta, cliente, defecto y metros
   rechazados
3. Sistema muestra un resumen previo: cuántos registros, qué rango de fechas, y
   qué defectos o líneas no reconoce
4. Usuario confirma y los registros quedan marcados como **semilla**

### Reglas de negocio
- **La semilla y los hallazgos son cosas distintas y no se mezclan.** El histórico
  registra merma consumada por pedido; un hallazgo registra algo detectado en el
  momento, que muchas veces evita la merma. Ambos alimentan el foco, pero se
  guardan por separado y se distinguen en pantalla.
- **La semilla se puede recargar.** Cargar el archivo otra vez reemplaza la
  semilla anterior, no la duplica. La clave es fecha, pedido y defecto.
- **La aplicación avisa cuando la semilla envejece.** Los datos vienen de
  papeletas que se llenan a mano y se transcriben, así que si nadie recarga, el
  foco se queda hablando del año pasado. Si el registro más reciente tiene más de
  cierto tiempo, se muestra desde cuándo no se actualiza.
- **Los defectos del archivo deben corresponder al catálogo de 40.** Los que no
  se reconozcan se reportan y no se cargan, en vez de entrar como categorías
  nuevas que ensucian el análisis.

### Sobre la papeleta
Hoy la merma se anota a mano en papeletas y alguien la transcribe a Excel. Esta
spec **no cambia ese proceso**: la aplicación consume el resultado.

Que la aplicación llegue a sustituir la papeleta es la evolución natural, pero es
otro alcance: toca a producción, no solo a calidad, y conviene decidirlo cuando
esta versión ya esté en uso. Anotarlo aquí evita que se cuele sin querer.

---

# SPEC-009 — Identidad y personal desde la suite

**Estado:** pendiente

### Reglas de negocio
- La lista de personal es `colaboradores` de la suite. Esta aplicación la lee y
  nunca la escribe.
- **Desaparece `imd_personal_procesos`**, la copia propia de la versión anterior.
- **Los registros guardan copia, no referencia:** una inspección conserva la
  nómina y el nombre tal como estaban al hacerla.

---

# SPEC-010 — Datos propios

**Estado:** pendiente

Inspecciones, hallazgos, acciones correctivas, catálogos, matriz de causas y
criterios de muestreo se quedan en el proyecto propio. Es la regla 4 de la
suite: la cuota del plan gratuito es por proyecto.

---

# Trabajo pendiente de definir

- **La matriz defecto ↔ variable** (SPEC-005), en junta de calidad, producción e
  ingeniería de procesos.
- **El criterio de muestreo** (SPEC-006): niveles, valores de AQL y si el cambio
  de severidad se automatiza.
- **El umbral de historia** a partir del cual se afina el foco por etiqueta
  (SPEC-004).
- **Cada cuánto se recarga la semilla de merma** y quién es responsable de
  hacerlo (SPEC-011). Sin eso, el aviso histórico envejece sin que nadie lo note.
- **Si la ventana de tres meses es fija o configurable.** Se definió en tres
  meses; queda por decidir si Calidad puede ajustarla sin tocar código.
- **A partir de cuántos registros se considera que una línea tiene poca
  actividad** y hasta dónde se amplía su ventana (SPEC-004). Lo define Calidad.
- **Quiénes son los inspectores de calidad** y quiénes los supervisores que
  atienden, para asignar `apps` y `roles.calidad` en la suite.
