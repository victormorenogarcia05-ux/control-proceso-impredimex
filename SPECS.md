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

**Los defectos se concentran y varían por máquina.** De 304 hallazgos, defectos
de impresión y tonos son el 73%. Pero el reparto cambia: en la RT7 los defectos
de impresión son el 76% y los tonos el 5%; en la RT6 van casi empatados, 47% y
45%; en la FL2 los tonos son el 56%. Eso confirma que el foco debe ser por
máquina, no general.

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

### Reglas de negocio
- **El foco es por máquina, no general.** Los datos muestran que el reparto de
  defectos difiere mucho entre equipos.
- **La propuesta es una guía, no una restricción.** El inspector puede revisar
  cualquier variable; la aplicación resalta las que más rinden.

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
El contenido. Basta empezar por los defectos que concentran el grueso:
**defectos de impresión** y **tonos**, que son el 73% de lo reportado.

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

### Qué se hereda
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
- **El histórico de defectos de 2025 y 2026**, seccionado por máquina, etiqueta y
  cliente. Puede cambiar el modelo: si los defectos se concentran por etiqueta o
  por cliente y no solo por máquina, el foco que se le muestra al inspector debe
  considerarlo.
- **Quiénes son los inspectores de calidad** y quiénes los supervisores que
  atienden, para asignar `apps` y `roles.calidad` en la suite.
