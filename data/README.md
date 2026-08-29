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

Cuatro tablas, todas unidas por `empleado_id` (600 empleados, 6 sedes):

| Archivo | Dimensiones | Contenido |
|---|---|---|
| `nimbus_empleados.csv` | 600 × 7 | atributos estables: sede, área, género, educación, antigüedad, grupo del piloto de fruta |
| `nimbus_bienestar_diario.csv` | 24.000 × 5 | panel diario de bienestar (escala 1-7) durante el piloto, línea de base + intervención |
| `nimbus_salario.csv` | 1.800 × 4 | panel de salario mensual por empleado y año (2023-2025), ejemplo de ajuste no lineal (edad-salario) |
| `nimbus_rrhh.csv` | 600 × 5 | **(agregada 24/08/2026 para la Clase 4)** renuncia (Sí/No) + tres señales de comportamiento |

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
| **URL desde el sitio** | `https://analiticadedatos-udesa.com/data/toy-nimbus/<archivo>.csv` — `nimbus_empleados.csv`, `nimbus_salario.csv`, `nimbus_bienestar_diario.csv` y `nimbus_rrhh.csv` (esta última agregada a `resources:` el 29/08/2026, al publicar la Clase 4). Los cuatro están declarados en `resources:` (`_quarto.yml`) y enlazados desde la página de la clase que los usa |
| Se usa en | Clase 1 (fundamentos de Python, piloto de fruta, ajuste no lineal salario~edad), Clase 2 (carga, `merge`, limpieza) y **Clase 4** (aprendizaje supervisado, KNN sobre `nimbus_rrhh.csv`) |

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
