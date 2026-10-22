# PARTE 1: P2 – Las 3 fallas del `Jenkinsfile` (2.0 pts)

Análisis del `Jenkinsfile` original: a continuación se explican las 3 fallas exactas y su corrección.

---

## Falla 1: Orden incorrecto de las etapas

**Ubicación:** `stage('Build image')` se ejecutaba **antes** de `stage('Regresion')`.

- **Qué estaba mal:** la imagen de Docker se construía antes de correr las pruebas de regresión.
- **Consecuencia:** el pipeline no funcionaba como un *Gate* verdadero (Deck 06/11). Si la regresión fallaba, la imagen ya había sido empaquetada y publicada erróneamente.
- **Corrección:** mover `stage('Regresion')` antes de `stage('Build image')`, de modo que solo se construya la imagen si las pruebas pasan.

---

## Falla 2: Supresión del código de salida con `|| true`

**Ubicación:** comando `bru run ... || true` en la etapa de regresión.

- **Qué estaba mal:** `|| true` forzaba un código de salida `0` aunque los tests de Bruno fallaran.
- **Consecuencia:** Jenkins interpretaba la etapa como exitosa. El pipeline nunca se marcaba en rojo (`FAILURE`) y avanzaba a pesar de tener tests fallidos.
- **Corrección:** eliminar `|| true` para que el código de salida real de `bru run` determine el resultado de la etapa.

---

## Falla 3: Ruta incorrecta del reporte JUnit

**Ubicación:** bloque `post { always }`.

- **Qué estaba mal:** el bloque buscaba `bruno/smoke/results.xml`, pero la etapa de regresión generaba el archivo en `bruno/regresion/results.xml`.
- **Consecuencia:** Jenkins publicaba reportes JUnit vacíos y el *Test Result* no mostraba qué peticiones o aserciones habían fallado.
- **Corrección:** actualizar la ruta a `bruno/regresion/results.xml`.

---

## Resumen

| # | Falla | Efecto | Corrección |
|---|-------|--------|------------|
| 1 | `Build image` antes de `Regresion` | Se publica imagen aunque falle la regresión | Reordenar etapas |
| 2 | `\|\| true` en `bru run` | Pipeline nunca falla | Quitar `\|\| true` |
| 3 | Ruta `smoke` en lugar de `regresion` | Reporte JUnit vacío | Corregir ruta a `bruno/regresion/results.xml` |

SHAs
7e8aa39