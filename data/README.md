# data/ — datasets de la materia

> Regla (`CONVENTIONS.md` §9): solo datasets chicos, con fuente y licencia documentadas
> acá. Las notebooks cargan los datos desde una **URL pública estable**; la copia local es
> para autoría/render.
>
> **Nota (2026-08-05):** este repo **pasó a privado**, así que su `raw.githubusercontent.com`
> ya no sirve para cargar datos (da 404 sin autenticación). Quedan dos fuentes públicas
> válidas, y ninguna URL de carga debe apuntar a este repo:
>
> - el **espejo público** `tomdamelio/analitica_de_datos_alumnos` vía
>   `raw.githubusercontent.com` — lo que usan hoy las notebooks (ver `TRAMPAS.md`);
> - el **sitio publicado**, `https://analiticadedatos-udesa.com/<ruta del archivo>`, para
>   los archivos declarados en `resources:` de `_quarto.yml`.
>
> Una nota anterior afirmaba lo contrario ("el repo es público, el raw sirve directo");
> quedó desactualizada al migrar el sitio a la VPS y cerrar el repo.

## `toy-nimbus/` — DATASET ESPINA de la materia (people analytics ficticio)

Dataset **sintético**, generado por `data/generar_toy_nimbus.py` (semilla fija, reproducible),
para acompañar el caso narrativo de la materia: Nimbus, una empresa de software ficticia con
alta rotación de personal, que corre un piloto de fruta gratis y se pregunta si puede predecir
la renuncia.

> **Es el dataset espina de la materia y se reutiliza en todas las clases** (decisión del
> docente, 24/08/2026). Antes la espina era `hr_attrition.csv` y Nimbus era exclusivo de la
> Clase 1. Ver `CONVENTIONS.md` §9.

Cinco tablas, todas unidas por `empleado_id` (600 empleados, 6 sedes):

| Archivo | Dimensiones | Contenido |
|---|---|---|
| `nimbus_empleados.csv` | 600 × 7 | atributos estables: sede, área, género, educación, antigüedad, grupo del piloto de fruta |
| `nimbus_bienestar_diario.csv` | 24.000 × 5 | panel diario de bienestar (escala 1-7) durante el piloto, línea de base + intervención |
| `nimbus_salario.csv` | 1.800 × 4 | panel de salario mensual por empleado y año (2023-2025), ejemplo de ajuste no lineal (edad-salario) |
| `nimbus_rrhh.csv` | 600 × 5 | **(agregada 24/08/2026 para la Clase 4)** renuncia (Sí/No) + tres señales de comportamiento |
| `nimbus_clima.csv` | 600 × 20 | **(agregada 01/09/2026 para la Clase 5)** encuesta anual de clima 2026: índice `bienestar_laboral` (0-100) + 19 predictores |

> ⚠️ **Hay dos "bienestar" y NO son lo mismo.** Es deliberado, pero se confunden fácil:
>
> | | `nimbus_bienestar_diario.csv` | `nimbus_clima.csv` |
> |---|---|---|
> | Qué mide | el piloto de fruta, día a día | encuesta anual de clima |
> | Escala | Likert 1-7, entero | índice 0-100, un decimal |
> | Filas | una por empleado y día | una por empleado |
> | Año | 2025 | 2026 |
> | ¿Se puede predecir? | **No.** Por diseño depende solo del tratamiento | **Sí**, esa es su razón de ser |
>
> El bienestar diario está construido para **inferencia causal**: no correlaciona con ninguna
> otra variable de Nimbus (con el salario da r = −0,019, R² = 0,0004). Intentar predecirlo con
> una regresión no da nada, y eso es una propiedad del diseño, no un defecto.

> **Por qué `nimbus_rrhh.csv` es una tabla aparte y no columnas nuevas en
> `nimbus_empleados.csv`:** la Clase 2 ya está publicada y tiene un ejercicio (celda 37) que
> pregunta *"`empleados` tiene 7 columnas y `bienestar` tiene 5, ¿cuántas tendrá la unión?"*.
> Agregarle columnas a `empleados` rompería ese ejercicio. Como beneficio lateral, la Clase 4
> arranca haciendo el `merge` que la Clase 2 enseñó.
>
> Por el mismo motivo, `generar_rrhh()` usa un `Generator` **propio** (`SEED_RRHH = 43`) en
> vez del `rng` encadenado del resto del script: si tomara números de ese stream, correría
> todos los sorteos posteriores y los otros tres CSV dejarían de ser idénticos. Verificado
> por hash el 24/08/2026: los tres originales no cambiaron ni un byte.

| Campo | Valor |
|---|---|
| Licencia | dataset propio, sintético — sin restricciones de uso |
| Generado | 2026-08 (regenerar con `python data/generar_toy_nimbus.py`) |
| **URL de carga en notebooks** | espejo público: `https://raw.githubusercontent.com/tomdamelio/analitica_de_datos_alumnos/main/data/toy-nimbus/<archivo>.csv` |
| **URL desde el sitio** | `https://analiticadedatos-udesa.com/data/toy-nimbus/<archivo>.csv` — `nimbus_empleados.csv`, `nimbus_salario.csv`, `nimbus_bienestar_diario.csv`, `nimbus_rrhh.csv` (agregada el 29/08/2026, al publicar la Clase 4) y `nimbus_clima.csv` (agregada el 02/09/2026, al publicar la Clase 5). Los cinco están declarados en `resources:` (`_quarto.yml`) y enlazados desde la página de la clase que los usa |
| Se usa en | Clase 1 (fundamentos de Python, piloto de fruta, ajuste no lineal salario~edad), Clase 2 (carga, `merge`, limpieza), Clase 3 (visualización), **Clase 4** (KNN sobre `nimbus_rrhh.csv`) y **Clase 5** (regresión lineal y regularización sobre `nimbus_clima.csv`) |

### `nimbus_clima.csv` — encuesta de clima laboral 2026 (Clase 5)

Agregada el 01/09/2026, tabla aparte y con `Generator` propio (`SEED_CLIMA = 44`), por el
mismo motivo que `nimbus_rrhh.csv`. **Verificado por hash: los cuatro CSV anteriores no
cambiaron ni un byte.**

Existe porque el bienestar del piloto de fruta **no se puede predecir** (ver el aviso de
arriba), y la Clase 5 necesita una variable continua que sí tenga estructura. El salario no
está en esta tabla sino en `nimbus_salario.csv`: armar el modelo exige un `merge`, que es el
paso que enseñó la Clase 2.

| Columna | Tipo | Rol en la clase |
|---|---|---|
| `empleado_id` | int | clave |
| `bienestar_laboral` | float (0-100) | **variable objetivo**: media 57,4, sd 11,2, rango 11,4-97,1 |
| `horas_extra_semana` | float | predictor real, efecto negativo. Tres casos en 38 h: **alto leverage** (el p99 es 18,5) |
| `apoyo_equipo` | float (1-10) | predictor real, el más fuerte después del salario |
| `reconocimiento` | float (1-10) | predictor real; **comparte factor latente con `apoyo_equipo`** |
| `autonomia` | float (1-10) | predictor real, efecto chico |
| `dias_home_office` | int (0-5) | **efecto no lineal**: óptimo en 3 días, U invertida |
| `bono_anual_pct` | float | **colineal con el salario** (r = 0,966), sin efecto propio |
| 12 columnas más | varios | **ruido puro**: coeficiente verdadero exactamente cero |

Las doce de ruido son `reuniones_semana`, `mensajes_chat_dia`, `dias_vacaciones_tomados`,
`distancia_oficina_km`, `cursos_completados`, `meses_en_el_rol`, `tickets_cerrados_mes`,
`emails_enviados_dia`, `proyectos_activos`, `dias_licencia_medica`, `horas_capacitacion` y
`puntualidad_pct`. Son doce y no dos a propósito: con pocas variables de ruido, OLS aguanta
bien incluso con 40 filas y la regularización se queda sin nada que arreglar.

#### Qué sección sostiene cada pieza (verificado 01/09/2026)

Con el salario expresado **en millones** (rango 0,94-1,52), que es la unidad que deja los
coeficientes legibles.

| Sección | Evidencia en los datos |
|---|---|
| Regresión simple | `bienestar = −31,04 + 70,30 · salario_en_millones`; R² = 0,327, RSE = 9,22, r = 0,572 |
| Regresión múltiple | salario 0,327 · apoyo 0,160 · reconocimiento 0,076; **juntos 0,522, no 0,563** |
| Colinealidad | β del bono: **+1,845 solo → +0,010** al agregar el salario |
| Interacción | β del producto = +0,437. Pendiente de horas extra: **−1,67 con apoyo bajo, −0,26 con apoyo alto** |
| No linealidad | home office: 0 d → 53,4 · **3 d → 60,9** · 5 d → 53,7 |
| Varianza no constante | sd de residuos 8,05 vs. 10,29. **Breusch-Pagan p = 8,7·10⁻⁵** sobre el modelo completo |
| Errores correlacionados | residuo medio por sede: de −2,55 (Córdoba) a +2,78 (Mar del Plata) |
| Outliers y leverage | 4 residuos con \|z\| > 3; 3 personas con 38 h extra |
| Inferencia (β significativo) | salario p = 1,8·10⁻⁵³; **ninguna de las 12 de ruido da p < 0,05** |
| Sobreajuste train/test | 6 variables reales: 0,631/0,586. Con las 19: 0,641/**0,557** |
| Sobreajuste polinómico | grado 1 test 0,40 · grado 5 test −262 · grado 9 train 0,634 / **test −359.441** |
| Dos ajustes que se dan vuelta | con `random_state=173` y 25 filas: recta 0,146/**0,289**, curva de grado 4 0,421/**−236,7** |

#### Ridge y lasso, promedio de 3 particiones

| Filas de entrenamiento | OLS | Ridge | Lasso |
|---|---|---|---|
| 30 | **−0,018** | 0,333 | 0,380 |
| 40 | 0,184 | 0,338 | 0,367 |
| 60 | 0,405 | 0,417 | 0,441 |
| 100 | 0,497 | 0,502 | 0,530 |
| 200 | 0,531 | 0,526 | 0,545 |

La ventaja de regularizar **se achica a medida que hay más datos**. Eso también es parte de la
lección, así que a propósito no se chequea al revés.

#### Regresión lineal vs. KNN (ISLP §3.5)

Partición 70/30, mejor K entre 3 y 80:

| Escenario | Lineal | KNN | Gana |
|---|---|---|---|
| 1 predictor, relación lineal (salario) | 0,267 | 0,270 | empatan |
| 1 predictor, relación no lineal (home office) | 0,003 | 0,056 | **KNN**, por lejos |
| 19 predictores, 12 de ruido | **0,557** | 0,401 | **Lineal** |

La tercera fila es la maldición de la dimensionalidad, y es el argumento de por qué en la
práctica se usa regresión lineal. La primera es honesta: cuando la relación es lineal empatan,
y KNN no da un coeficiente que se pueda interpretar.

#### Cuatro avisos para quien arme la clase

1. **El lasso y la colinealidad.** Como el bono y el salario tienen r = 0,966, el lasso a veces
   se queda con uno y apaga el otro casi arbitrariamente. No es un bug: es lo que hace el lasso
   ante predictores colineales, y ridge en cambio los reparte. Conviene decirlo en la diapo de
   Ridge vs. Lasso en vez de que aparezca de sorpresa.
2. **La superficie de RSS sobre (β₀, β₁) es un cañón, no un tazón.** Con el salario crudo el
   valle es 801 veces más largo que ancho, y aun centrado es 120 veces. Para la animación 3D o
   de contornos hay que **estandarizar el predictor**, o no se ve ningún mínimo. Centrar tiene
   además un beneficio: β₀ pasa de −31,04 (extrapolación sin sentido, nadie cobra cero) a
   57,36, que es el bienestar de quien cobra el salario promedio.
3. **El R² en entrenamiento nunca baja al agregar variables.** Es matemático. Un ejercicio del
   tipo "agregá variables y mirá si el R² mejora o empeora" solo puede mostrar "empeora" **en
   testeo**: en entrenamiento la serie es monótona creciente (0,346 → 0,633 agregando diez).
4. **El embudo de heterocedasticidad es sutil a ojo** (razón 1,28 entre mitades). No se puede
   hacer más marcado sin romper la escala 0-100 y hundir el R² de la regresión simple, así que
   para ilustrar el concepto conviene una figura esquemática.

El piloto de fruta de 2025 **no tiene efecto** sobre este índice (p = 0,647), a propósito: si lo
tuviera, ensuciaría la historia causal de la Clase 1.

### `nimbus_rrhh.csv` — renuncia y señales de comportamiento (Clase 4)

Agregada el 24/08/2026. Una fila por empleado.

| Columna | Tipo | Qué es |
|---|---|---|
| `empleado_id` | int | clave, une con las otras tres tablas |
| `faltas_mes` | int | días de ausencia en el último mes |
| `weeklys_perdidas` | int (0-12) | weeklys del trimestre a las que no asistió |
| `minutos_camara_weekly` | float (0-45) | promedio de minutos con la cámara encendida |
| `renuncia` | "Si"/"No" | **variable objetivo** (17,0% "Si") |

**La estructura causal es deliberada, y es el material didáctico de la Clase 4.**

La cadena es **nivel educativo → salario → renuncia**. El salario es *la causa*: cobrar poco
empuja a buscar otro trabajo. El nivel educativo **no tiene efecto propio**: influye en la
renuncia sólo a través del salario, o sea que su efecto está mediado. A eso se suman el grupo
del piloto de fruta y la antigüedad.

Aparte están las **señales de comportamiento**, que son **consecuencia** del riesgo latente,
no causa: quien ya se está por ir falta más y se desengancha de las weeklys. Por eso predicen
muy bien y no explican nada, que es justo el contraste que la clase necesita.

| | Explican | Predicen |
|---|---|---|
| **Causas**: salario, fruta, antigüedad (+ educación, mediada) | sí, con signo interpretable | mal |
| **Consecuencias**: faltas, weeklys perdidas, minutos de cámara | no, son síntomas | muy bien |

Efectos verificados por `verificar_rrhh()` en cada corrida del generador:

| Efecto | Valor |
|---|---|
| **Salario (la causa)** | se van 1.181.539 vs. se quedan 1.273.157 |
| Nivel educativo (mediado por salario) | corr = −0,251: a más educación, menos renuncias |
| Piloto de fruta | Tratamiento 15,1% vs. Control 18,9% |
| Faltas/mes | se van 2,63 vs. se quedan 1,10 |
| Weeklys perdidas | se van 4,13 vs. se quedan 1,47 |
| Minutos de cámara | se van 23,4 vs. se quedan 35,1 |

Poder predictivo con KNN (30% de testeo, baseline "nadie renuncia" = 83,0%):

| Predictores | K=1 (train/test) | mejor test |
|---|---|---|
| Señales de comportamiento | 1,000 / 0,861 | **0,911** (K=51) |
| Solo `faltas_mes` + `minutos_camara_weekly` | 0,979 / 0,878 | 0,911 (K=11) |
| Causas (salario, fruta, antigüedad, educación) | 0,998 / **0,756** | 0,833 (K=21) |

> ⚠️ **Dos cosas a tener presentes al usar esta tabla en clase.**
>
> 1. **Con 17% de renuncias, la exactitud es una métrica floja**: predecir que no renuncia
>    nadie ya da 83,0%. El modelo con las causas llega a 0,833, o sea que **no le gana al
>    modelo trivial**, y con K=1 da 0,756, bastante *peor*. Es didácticamente útil (es el
>    mejor argumento para mostrar la línea de base), pero conviene mostrarla explícitamente.
>    Precisión y exhaustividad se ven recién en la Clase 6.
> 2. **La renuncia por nivel educativo no es perfectamente monótona**: Secundario 25,0% y
>    Terciario 31,5% aparecen invertidos, porque Secundario tiene sólo ~52 personas y es
>    ruidoso. La tendencia global sí se sostiene (corr = −0,251). Si se muestra un gráfico de
>    barras por educación, conviene saberlo de antemano.

**Publicación:** para que las notebooks de la Clase 4 puedan cargarlo desde Colab, este
archivo tiene que quedar declarado en `resources:` de `_quarto.yml` y llegar al espejo
público. Pendiente al 24/08/2026.

**Piloto de fruta (`nimbus_bienestar_diario.csv`).** Randomizado por sede, no por persona:
Buenos Aires, Mendoza y Rosario reciben fruta gratis (`grupo_fruta = "Tratamiento"`); Bahía
Blanca, Córdoba y Mar del Plata no (`"Control"`). El bienestar promedio en el grupo tratado es
mayor de forma estadísticamente significativa (diferencia chica, ~0.34 puntos en escala 1-7,
pero el panel tiene 24.000 filas — por eso el p-valor es minúsculo pese al efecto chico).

**Salario (`nimbus_salario.csv`).** El salario depende de la edad con un efecto cuadrático
(pico ~46 años) y de la educación, más ruido — pensado como ejemplo de "ajuste" tipo Wage de
ISLP cap. 1, pero con datos propios de Nimbus. No depende del género (decisión deliberada: no
hay justificación pedagógica para modelar una brecha salarial en un dataset de juguete).

**Ilustrativo, no real:** las slides de la Clase 1 sobre % de renuncia vs. salario, accuracy de
clasificación por variable, regresión/clasificación y clustering usan datos **sintéticos
generados en el cliente** (no de este dataset) porque no existe una variable de
renuncia/attrition en `toy-nimbus` — está aclarado en las notas del orador de esas slides.

## `student_dropout.csv` — deserción y éxito académico (Clase 10)

**Predict Students' Dropout and Academic Success** — dataset real de una institución de
educación superior (Portugal), recopilado por Realinho, Vieira Martins, Machado & Baptista
(2021) con fines de investigación sobre deserción estudiantil.

| Campo | Valor |
|---|---|
| Dimensiones | 4.424 filas × 37 columnas (1 fila = 1 estudiante) |
| Variable objetivo | `Target` (Dropout 1.421 / Graduate 2.209 / Enrolled 794) |
| Faltantes | 0 |
| Fuente primaria | UCI Machine Learning Repository, dataset **id 697** ([enlace](https://archive.ics.uci.edu/dataset/697/predict+students+dropout+and+academic+success)) |
| **URL de carga en notebooks** | `https://archive.ics.uci.edu/static/public/697/predict+students+dropout+and+academic+success.zip` (archivo `data.csv`, separador `;`) |
| Licencia | **CC BY 4.0** (permite uso y redistribución con atribución) |
| Descargado | 2026-07-19 |
| Se usa en | Clase 10 (PCA: motivación, intuición 2D/3D, biplot, escalado, scree) |

**Uso pedagógico en la Clase 10.** Es la base de datos que acompaña toda la explicación de
PCA. Sobre su bloque de variables académicas numéricas (materias inscriptas/aprobadas y notas
por semestre, nota de admisión, edad), PC1 explica ~54% de la varianza y separa nítidamente a
quienes desertan de quienes se gradúan: la primera componente resulta ser un eje de riesgo
académico. El ejercicio de cierre de la clase aplica el método a `hr_attrition.csv`, que los
estudiantes ya conocen. La cita de atribución (CC BY 4.0): Realinho, V., Vieira Martins, M.,
Machado, J. & Baptista, L. (2021). *Predict Students' Dropout and Academic Success*. UCI ML
Repository. https://doi.org/10.24432/C5MC89

## `hr_attrition.csv` — material histórico (ya no es la espina)

> **Dejó de ser el dataset espina el 24/08/2026** (ver `toy-nimbus/` arriba). Sigue acá
> porque la Clase 2 lo usa en un tramo, pero **no se propone por defecto** para clases nuevas.

**IBM HR Analytics Employee Attrition & Performance** — dataset ficticio creado por
científicos de datos de IBM para ilustrar problemas de people analytics (predicción de
rotación de personal / *turnover*).

| Campo | Valor |
|---|---|
| Dimensiones | 1.470 filas × 35 columnas (1 fila = 1 empleado/a) |
| Variable objetivo | `Attrition` (Yes/No; 237 Yes = 16,1%) |
| Faltantes / duplicados | 0 / 0 (dataset "limpio de fábrica"; ver nota pedagógica) |
| Fuente primaria | IBM Sample Data. Publicado por IBM en su repo oficial [`IBM/employee-attrition-aif360`](https://github.com/IBM/employee-attrition-aif360) (archivo `data/emp_attrition.csv`) |
| Fuente de difusión | [Kaggle: IBM HR Analytics Attrition](https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset) (requiere login; NO se usa para cargar) |
| **URL de carga en notebooks** | `https://raw.githubusercontent.com/IBM/employee-attrition-aif360/master/data/emp_attrition.csv` |
| Descargado | 2026-07-07 |
| Se usa en | Clase 2 (EDA/limpieza), y previsto para 3 (viz), 4–7 (supervisado), 9–10 (no supervisado) |

### Licencia — ⚠️ punto a validar por la cátedra

- La versión difundida en Kaggle figura como *"Database: Open Database, Contents:
  Database Contents"*, pero IBM nunca publicó una licencia abierta formal específica
  para el dataset. Es de uso educativo **muy** extendido.
- El repo **oficial de IBM** que lo contiene (`IBM/employee-attrition-aif360`) está
  licenciado **Apache-2.0**, lo que cubre el repo y razonablemente sus datos de ejemplo;
  aun así, la licencia del *repo* no es una declaración explícita sobre el *dataset*.
- **Decisión adoptada en Fase 1 (conservadora):** las notebooks cargan el CSV **desde la
  URL raw del repo de IBM** (fuente pública, oficial y estable) y **no** se sirve el CSV
  desde el sitio público de la materia. Si la cátedra decide que Apache-2.0 alcanza, se
  puede pasar a servirlo desde GitHub Pages (`<site-url>/data/hr_attrition.csv`) como
  respaldo ante cambios en el repo de IBM. Registrado como pregunta abierta en `REPORT.md`.

### Nota pedagógica (Clase 2)

El dataset original **no tiene faltantes ni duplicados**. Para enseñar detección y
manejo de datos faltantes/duplicados, la notebook de la Clase 2 introduce "suciedad"
**determinística y documentada** (reglas por índice, sin aleatoriedad, idénticas en
Python y R) sobre una copia, y luego la limpia. Los descriptivos y figuras del análisis
final salen siempre de los datos reales.

### Diccionario de variables (subset usado en Clase 2)

| Variable | Tipo | Descripción |
|---|---|---|
| `Age` | int | Edad en años |
| `Attrition` | cat (Yes/No) | Si el empleado dejó la empresa |
| `Department` | cat (3) | Departamento |
| `JobRole` | cat (9) | Puesto |
| `MonthlyIncome` | int | Ingreso mensual (USD) |
| `TotalWorkingYears` | int | Años de experiencia laboral total |
| `YearsAtCompany` | int | Antigüedad en la empresa |
| `DistanceFromHome` | int | Distancia casa–trabajo (km) |
| `OverTime` | cat (Yes/No) | Hace horas extra |
| `JobSatisfaction` | ord 1–4 | Satisfacción con el trabajo |
| `WorkLifeBalance` | ord 1–4 | Balance vida–trabajo |
| `EnvironmentSatisfaction` | ord 1–4 | Satisfacción con el ambiente |
| `MaritalStatus` | cat (3) | Estado civil |
| `NumCompaniesWorked` | int | Empresas anteriores |

Columnas constantes sin información (`EmployeeCount`, `StandardHours`, `Over18`) se
usan en la clase justamente como ejemplo de columnas a descartar.
