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

Once tablas. Diez se unen por `empleado_id` (600 empleados, 6 sedes); la restante,
`nimbus_soporte_diario.csv`, es una **serie diaria de la empresa** y se une por `fecha`:

| Archivo | Dimensiones | Contenido |
|---|---|---|
| `nimbus_empleados.csv` | 600 × 7 | atributos estables: sede, área, género, educación, antigüedad, grupo del piloto de fruta |
| `nimbus_bienestar_diario.csv` | 24.000 × 5 | panel diario de bienestar (escala 1-7) durante el piloto, línea de base + intervención |
| `nimbus_salario.csv` | 1.800 × 4 | panel de salario mensual por empleado y año (2023-2025), ejemplo de ajuste no lineal (edad-salario) |
| `nimbus_rrhh.csv` | 600 × 5 | **(agregada 24/08/2026 para la Clase 4)** renuncia (Sí/No) + tres señales de comportamiento |
| `nimbus_clima.csv` | 600 × 20 | **(agregada 01/09/2026 para la Clase 5)** encuesta anual de clima 2026: índice `bienestar_laboral` (0-100) + 19 predictores |
| `nimbus_modalidad.csv` | 600 × 2 | **(agregada 10/09/2026 para la Clase 6)** modalidad de trabajo, presencial o remoto, diseñada para reproducir el *confounding* de ISLP Fig. 4.3 |
| `nimbus_nivel.csv` | 600 × 2 | **(agregada 11/09/2026 para la Clase 6)** nivel del puesto, junior o senior, diseñada para el *confounding* de ISLP Fig. 4.3. **No es** `antiguedad_anios` |
| `nimbus_fruta.csv` | 600 × 2 | **(agregada 11/09/2026 para la Clase 6)** promedio de días por semana con fruta en la oficina (0-5, con un decimal), el predictor del repaso de regresión lineal. **No es** el piloto de la Clase 1 |
| `nimbus_soporte_diario.csv` | 1.096 × 5 | **(agregada 17/09/2026 para la Clase 7)** tres años de tickets diarios de la mesa de ayuda, con tendencia, dos estacionalidades semanales opuestas, estacionalidad anual, feriados y un incidente. **No tiene `empleado_id`**: es una serie de la empresa, no del empleado |
| `nimbus_desempeno.csv` | 600 × 12 | **(agregada 28/09/2026 para la Clase 9)** once indicadores de desempeño con dos factores latentes (cumplimiento y colaboración) en escalas mezcladas; el cumplimiento cae en quien renuncia |
| `nimbus_patrones.csv` | 600 × 10 | **(agregada 05/10/2026 para la Clase 10)** patrones de trabajo: seis métricas de huella digital y tres ítems de encuesta, con cuatro perfiles latentes (carga × enganche) que se cruzan con la renuncia |

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
| **URL desde el sitio** | `https://analiticadedatos-udesa.com/data/toy-nimbus/<archivo>.csv` — `nimbus_empleados.csv`, `nimbus_salario.csv`, `nimbus_bienestar_diario.csv`, `nimbus_rrhh.csv` (agregada el 29/08/2026, al publicar la Clase 4), `nimbus_clima.csv` (agregada el 02/09/2026, al publicar la Clase 5), `nimbus_soporte_diario.csv` (Clase 7) y `nimbus_desempeno.csv` (agregada el 29/09/2026, al publicar la Clase 9). Todos están declarados en `resources:` (`_quarto.yml`) y enlazados desde la página de la clase que los usa |
| Se usa en | Clase 1 (fundamentos de Python, piloto de fruta, ajuste no lineal salario~edad), Clase 2 (carga, `merge`, limpieza), Clase 3 (visualización), **Clase 4** (KNN sobre `nimbus_rrhh.csv`), **Clase 5** (regresión lineal y regularización sobre `nimbus_clima.csv`), **Clase 6** (regresión logística sobre `nimbus_rrhh.csv`, *confounding* con `nimbus_modalidad.csv`), **Clase 7** (series temporales sobre `nimbus_soporte_diario.csv`), **Clase 8** (árboles y random forest sobre `nimbus_rrhh.csv`), **Clase 9** (PCA sobre `nimbus_desempeno.csv`) y **Clase 10** (clustering sobre `nimbus_patrones.csv`, en preparación) |

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
generados en el cliente** (no de este dataset) porque cuando se armó la Clase 1 no existía una
variable de renuncia en `toy-nimbus` (hoy sí: `nimbus_rrhh.csv`, agregada para la Clase 4) —
está aclarado en las notas del orador de esas slides.

### `nimbus_modalidad.csv` — modalidad de trabajo (Clase 6)

Agregada el 10/09/2026, tabla aparte y con `Generator` propio (`SEED_MODALIDAD = 45`), por
el mismo motivo que las dos anteriores. **Verificado por hash: los cinco CSV anteriores no
cambiaron ni un byte.**

Existe para reproducir con Nimbus el *confounding* que ISLP cuenta con el dataset Default
(Fig. 4.3: los estudiantes tienen más deuda, por eso en promedio caen más en default, pero a
igual deuda caen menos). Pedido del docente: el ejemplo del libro pero con datos propios.

| Columna | Tipo | Rol en la clase |
|---|---|---|
| `empleado_id` | int | clave |
| `modalidad` | `presencial` / `remoto` (37,7% remoto) | el predictor cualitativo que se da vuelta |

**La estructura causal, que es el material de la slide:** los remotos prenden **menos** la
cámara (fatiga de videollamadas), y por eso, en promedio, **renuncian más**. Pero a **igual**
cámara, un remoto renuncia **menos** (la flexibilidad retiene). Como `renuncia` y
`minutos_camara_weekly` ya estaban fijos, la modalidad se sortea condicionada a las dos.

| | Presencial | Remoto |
|---|---|---|
| Minutos de cámara (media) | 35,3 | 29,7 |
| Renuncia | 12,8% | 23,9% |

| Modelo | β de remoto | p |
|---|---|---|
| `renuncia ~ modalidad` | **+0,76** | 0,0006 |
| `renuncia ~ modalidad + minutos_camara` | **−1,14** | 0,0013 |

Es la Tabla 4.2 contra la Tabla 4.3 de ISLP, con Nimbus. Verificado el 10/09/2026 sobre las
600 filas (`verificar_modalidad()` en el generador lo exige en cada corrida).

### `nimbus_nivel.csv` — nivel del puesto (Clase 6)

Agregada el 11/09/2026, tabla aparte y con `Generator` propio (`SEED_NIVEL = 47`). **Verificado
por hash: los CSV anteriores no cambiaron ni un byte.**

> ⚠️ **No confundir con `antiguedad_anios`** (en `nimbus_empleados.csv`). Eso son **años en la
> empresa**; esto es la **banda del puesto**. Se puede entrar como senior con un año de
> antigüedad.

| Columna | Tipo | Rol en la clase |
|---|---|---|
| `empleado_id` | int | clave |
| `nivel` | `junior` / `senior` (49,8 % juniors) | el predictor cualitativo que se da vuelta |

Reemplaza a `nimbus_modalidad.csv` como ejemplo de *confounding*: el docente descartó el par
modalidad/cámara el 11/09/2026 por poco creíble. La historia nueva es la del estudiante y la
tarjeta de ISLP, en lenguaje de RRHH: **los juniors ganan menos**, **cobrar poco predice
renunciar**, y por eso **en bruto los juniors renuncian más**. Pero **a igual salario el que se
va es el senior**, porque un senior que cobra lo que un junior está subpagado para su nivel.

Como `renuncia` y `salario_mensual` ya estaban fijos, el nivel se sortea condicionado a los dos.

| | Senior | Junior |
|---|---|---|
| Salario medio | 1,32 M | 1,20 M |
| Renuncia | 12,6 % | 21,4 % |

| Modelo | β de junior | p |
|---|---|---|
| `renuncia ~ nivel` | **+0,63** | 0,005 |
| `renuncia ~ nivel + salario` | **−1,34** | 0,0001 |

Es la Tabla 4.2 contra la Tabla 4.3 de ISLP, con Nimbus. `verificar_nivel()` lo exige en cada
corrida del generador.

**`nimbus_modalidad.csv` queda huérfana:** sigue en `data/` y documentada, pero desde el
11/09/2026 ninguna clase la lee.

### `nimbus_fruta.csv` — promedio de días con fruta en la oficina (Clase 6)

Agregada el 11/09/2026, tabla aparte y con `Generator` propio (`SEED_FRUTA = 46`), por el
mismo motivo que las tres anteriores. **Verificado por hash: los seis CSV anteriores no
cambiaron ni un byte.**

> ⚠️ **Hay dos "fruta" y NO son lo mismo.** `grupo_fruta` (en `nimbus_empleados.csv`) es el
> **piloto aleatorizado** de la Clase 1: binario, control contra tratamiento, pensado para
> inferencia causal. `dias_fruta_semana` (acá) es un **conteo observacional** de 2026, de la
> misma ventana que la encuesta de clima, promediado sobre un trimestre (de ahí el decimal). En la Clase 6 aparecen los dos: este en el repaso
> de regresión lineal, y `grupo_fruta` como predictor *dummy* de la logística.

| Columna | Tipo | Rol en la clase |
|---|---|---|
| `empleado_id` | int | clave |
| `dias_fruta_semana` | float 0-5, un decimal (media 2,49; 28 empleados en 0 exacto) | el predictor del repaso de regresión lineal |

Existe por un motivo didáctico puntual, pedido por el docente el 11/09/2026: que el
**intercepto signifique algo**. Hasta ese día el repaso era `bienestar ~ salario`, y ahí β₀
es "el bienestar de alguien que cobra cero", que da **−31** en una escala de 0 a 100 y no
describe a nadie. Con los días de fruta el cero **existe** (31 empleados) y β₀ se lee
directo. De yapa, la extrapolación se pasa de 100 (la recta cruza el techo recién en 9,2
días por semana, que no existe), que es el problema con el que abre el bloque siguiente de
la clase: una recta no respeta un rango.

Se sortea **condicionado** al `bienestar_laboral` que ya estaba fijo en `nimbus_clima.csv`
(una normal centrada en el bienestar del empleado, recortada a [0, 5]), así la relación
existe sin tocar una fila de las tablas anteriores. El recorte en 0 es el que deja gente en
el cero exacto, que es lo que hace que el intercepto describa a alguien. **En clase se presenta como asociación,
no como efecto causal**: el experimento es el de la Clase 1, este es una encuesta.

| | Valor |
|---|---|
| β₀ (intercepto) | **41,60** · bienestar de quien nunca tiene fruta |
| β₁ (por día) | **+6,34** |
| R² | 0,567 |
| p de los dos coeficientes | < 0,001 |

Verificado el 11/09/2026 sobre las 600 filas (`verificar_fruta()` en el generador lo exige en
cada corrida).

### `nimbus_soporte_diario.csv` — tickets diarios de la mesa de ayuda (Clase 7)

Agregada el 17/09/2026, tabla aparte y con `Generator` propio (`SEED_SOPORTE = 48`), por el
mismo motivo que las anteriores. **Verificado por hash: los ocho CSV anteriores no cambiaron
ni un byte.**

Existe porque el docente pidió dar series temporales **sin salir de Nimbus** (17/09/2026). La
alternativa era un dataset externo real (bicicletas compartidas de Washington DC, UCI, CC BY
4.0), que se descartó para no romper la continuidad del caso.

> ⚠️ **Hay dos series diarias de Nimbus y NO son lo mismo.** `nimbus_bienestar_diario.csv` es
> el panel del piloto de fruta: 40 días hábiles, Likert 1-7, **sin tendencia ni
> estacionalidad por diseño** (sirve para inferencia causal, no se puede pronosticar).
> `nimbus_soporte_diario.csv` es lo contrario: tres años de conteos diarios **con** toda la
> estructura temporal. Es la única tabla de Nimbus que se puede descomponer.

| Columna | Tipo | Rol en la clase |
|---|---|---|
| `fecha` | date, diaria y continua, 01/01/2023 a 31/12/2025 (1.096 días) | el índice de la serie |
| `tickets_empresas` | int | **la serie protagonista**: clientes corporativos, pico de lunes a viernes |
| `tickets_particulares` | int | usuarios individuales, el ritmo opuesto: pico el fin de semana |
| `tickets` | int | la suma de las dos |
| `feriado` | 0/1 | feriado nacional argentino de fecha fija |

**Lo que tiene adentro, y para qué está cada cosa** (todo verificado el 17/09/2026; los
asserts de `verificar_soporte_diario()` lo exigen en cada corrida):

| Estructura | Número | Para qué |
|---|---|---|
| Tendencia | 107 → 145 tickets/día (empresas), 215 → 294 (total) de 2023 a 2025 | que la serie crezca, y que un modelo sin tendencia falle al extrapolar |
| Estacionalidad semanal, empresas | lunes 176 · domingo 28 (fin de semana = **0,21×** el día hábil) | es lo que hace ganar al naive estacional |
| Estacionalidad semanal, particulares | domingo 242 · miércoles 76 (fin de semana = **2,72×**) | el ritmo opuesto |
| Estacionalidad semanal del **total** | entre 241 y 270: **swing de 11%** | **el punto de la clase**: dos ritmos opuestos de tamaño parecido casi se cancelan. La serie agregada parece no tener patrón semanal y en realidad tiene dos. Agregar borró el comportamiento |
| Estacionalidad anual | empresas: enero 93, agosto 160 | enero es vacaciones en Argentina |
| Feriados | caen al 35% si son día hábil; dejan un resto medio de **−104** | la descomposición no maneja sola los efectos de calendario (FPP3 §3.6) |
| Incidente del 14/08/2024 | pico de ×3,2, resto de **+398** (el más grande de la serie) | el outlier que justifica STL robusto, y que muestra que el resto no es basura: es donde quedan las noticias |

**Descomposición STL** (`period=7`, `robust=True`) sobre `tickets_empresas`: fuerza estacional
**0,869**, tendencia de 78 a 129. Los cinco restos más grandes son el incidente (dos días) y
tres feriados.

**Comparación de métodos de referencia** (entrenar hasta el 30/09/2025, pronosticar los 31
días de octubre de 2025), sobre `tickets_empresas`:

| Método | MAE | RMSE | MAPE | MASE |
|---|---|---|---|---|
| Media | 73,4 | 75,8 | 83,3% | 1,43 |
| Naive | 64,9 | 94,4 | 130,3% | 1,26 |
| Drift | 68,0 | 96,7 | 133,5% | 1,32 |
| **Naive estacional (m = 7)** | **17,3** | **22,0** | **12,8%** | **0,34** |

Y el giro que cierra el círculo con la estructura: **sobre el total, el naive estacional
pierde** (MAE 26,5) contra el naive simple (14,2), justamente porque el total casi no tiene
estacionalidad semanal. El método que gana depende de la estructura que la serie tiene, y esa
estructura se vio al descomponerla.

De yapa, el MAPE del naive (130%) es la demostración de su propia trampa: los domingos tienen
pocos tickets, el denominador se achica y el porcentaje explota.


### `nimbus_desempeno.csv` — indicadores de desempeño (Clase 9)

Once indicadores por empleado que RRHH junta de distintos sistemas. Existen para la clase de
PCA: un set grande de variables de desempeño que se reduce a **una** métrica, y esa métrica
predice la renuncia, lo que empalma PCA con el aprendizaje supervisado de las clases 4–8.

| Factor latente | Indicadores (escala) |
|---|---|
| **Cumplimiento** (cae en quien renuncia; sube un poco con la antigüedad) | `objetivos_cumplidos_pct` (0-100), `entregas_a_tiempo_pct` (0-100), `tareas_cerradas_mes` (conteo), `horas_foco_semana` (horas), `evaluacion_lider` (1-5), `okr_score` (0-1), `retrabajos_mes` (conteo, **carga negativa**) |
| **Colaboración** (no depende de la renuncia; sube un poco con la antigüedad) | `evaluacion_pares` (1-5), `minutos_mentoria_mes` (minutos, desvío ~83), `revisiones_a_otros_mes` (conteo), `iniciativas_internas_anio` (conteo) |

Relato causal, coherente con `nimbus_rrhh.csv`: el cumplimiento es un **síntoma** del
desenganche, como las faltas y la cámara. Predice la renuncia, no la explica. Como `renuncia`
ya estaba fija, el factor se genera condicionado a ella (semilla propia, `SEED_DESEMPENO = 49`).
Las medidas acotadas (porcentajes, notas 1-5, OKR 0-1) se llevan a su escala con una curva
logística y no con un recorte: así se comprimen de a poco cerca del techo, como un porcentaje real,
en vez de apilarse en 100 % (cambio del 28/09/2026, pedido al ver la nube en la slide de PC1).

**Efectos verificados** (28/09/2026; los asserts de `verificar_desempeno()` los exigen en cada
corrida). PCA sobre los indicadores estandarizados:

| Qué | Valor | Qué sostiene en la clase |
|---|---|---|
| PVE de PC1 / PC2 / PC3 | **40,4% / 21,7% / 5,8%** | dos componentes interpretables y un codo claro en la tercera |
| Cargas de PC1 | 0,33-0,41 en los siete de cumplimiento (retrabajos con signo opuesto), ≤ 0,04 en colaboración | PC1 = índice de cumplimiento |
| Cargas de PC2 | 0,46-0,52 en los cuatro de colaboración, ≤ 0,04 en cumplimiento | PC2 = colaboración |
| PCA **sin** estandarizar | PC1 explica el 94,6% y es `minutos_mentoria_mes` sola | el contraejemplo de no escalar |
| AUC de renuncia (logística, CV 5×10) | 11 indicadores **0,828**; PC1 sola **0,832**; 2-11 componentes 0,828-0,831; PC1 sin escalar **0,464**; PC2 sola 0,506 | una sola componente comprime once variables sin perder poder predictivo; elegir `n_components` por CV da una meseta desde 1 |
| Correlación de PC1 con `nimbus_rrhh` | faltas −0,18, weeklys perdidas −0,31, minutos de cámara +0,30; antigüedad +0,21 | es otro síntoma del mismo desenganche |

> **Aviso para quien arme la clase:** que PC1 prediga la renuncia es una **propiedad de estos
> datos**, no de PCA. PCA no mira `y`: la componente de más varianza podría no tener nada que
> ver con el objetivo (ISLP §6.3.1). En Nimbus coincide porque el factor dominante es,
> por diseño, el que cae antes de renunciar.


### `nimbus_patrones.csv` — patrones de trabajo (Clase 10)

Agregada el 05/10/2026, tabla aparte y con `Generator` propio (`SEED_PATRONES = 50`). **Verificado
por hash: los diez CSV anteriores no cambiaron ni un byte.**

Existe porque ninguna tabla anterior de Nimbus tiene grupos: con K-means sobre desempeño, señales
de RRHH, clima o edad–salario, el silhouette da entre 0,13 y 0,43, siempre máximo en K=2 y
decreciente después. Son nubes de una sola pieza. Pedido del docente: datos donde el clustering
encuentre **perfiles de empleados con sentido**.

Hay cuatro perfiles latentes en un 2×2 de **carga × enganche**. **El perfil no está en el CSV**:
es lo que el alumno tiene que descubrir. Vive solo dentro del generador, para los asserts.

| Perfil | Firma | Renuncia |
|---|---|---|
| Comprometidos | muchas horas, mucho compromiso, red de contactos amplia | ~5 % |
| Quemados | más horas todavía, horas fuera de horario, mucho agotamiento | ~28 % |
| Desenganchados | pocas horas, poco compromiso, pocos mensajes y contactos | ~31 % |
| Nuevos | pocas horas, mucho compromiso, **mucha capacitación**; antigüedad ≤ 2 años | ~7 % |

El perfil se sortea **condicionado a la `renuncia`** de `nimbus_rrhh.csv` (ya fija) y a la
antigüedad, igual que `nimbus_desempeno`. **La renuncia no entra al clustering**: se cruza
después, como validación externa.

| Columna | Fuente | Rol en la clase |
|---|---|---|
| `empleado_id` | — | clave |
| `horas_semana` | huella | **eje del plano ancla** (carga) |
| `indice_compromiso` | encuesta, 0-100 | **eje del plano ancla** (enganche) |
| `horas_fuera_horario_semana` | huella | marca a los Quemados |
| `minutos_reunion_semana` | huella | **en minutos a propósito**: sin estandarizar se come la distancia |
| `mensajes_dia` | huella | bajo en Desenganchados |
| `contactos_distintos_mes` | huella | alto en Comprometidos; bajo en Nuevos, que todavía arman su red |
| `horas_capacitacion_mes` | huella | lo que distingue a los Nuevos |
| `agotamiento` | encuesta, 1-7 | alto en Quemados |
| `recomendaria_0a10` | encuesta, entero (tipo eNPS) | bajo en Quemados y Desenganchados |

**La estructura es anidada a propósito.** En el plano ancla, la brecha de compromiso entre
{Comprometidos, Nuevos} y {Quemados, Desenganchados} es mayor que la de horas dentro de cada par.
Por eso K=2 da "enganchados contra no enganchados" y K=4 da los perfiles, y el dendrograma tiene
una jerarquía real.

**Efectos verificados** (05/10/2026; los asserts de `verificar_patrones()` los exigen en cada
corrida). K-means con `n_init=10` y `random_state=50`:

| Qué | Valor | Qué sostiene en la clase |
|---|---|---|
| Plano ancla estandarizado, silhouette K=2..8 | 0,448 · 0,545 · **0,558** · 0,477 · 0,426 · 0,374 · 0,358 | K=2/3/4 sobre el mismo plano; el máximo en K=4 |
| Plano ancla, ARI de K=4 contra el perfil latente | **0,908** | separable, pero con solapamiento creíble |
| Mínimos locales (`init="random"`, `n_init=1`, semillas 0-49) | óptimo W = 187,2 en 46 semillas; **W ≈ 308,5 en las semillas 0, 2, 32 y 36** (parten un grupo de abajo y juntan los dos de arriba) | la animación de semillas (ISLP Fig. 12.8) |
| Las 9 variables estandarizadas, silhouette K=3 / K=4 | 0,367 / **0,370**; ARI de K=4 0,991 | en p dimensiones K=4 gana por poco: el codo es más claro que el silhouette |
| Sin estandarizar | ARI **0,068**; los clusters son niveles de `minutos_reunion_semana` (128 · 438 · 723 · 1.065 min; η² = 0,906) | la trampa de escala (ISLP Fig. 12.15) |
| Renuncia por cluster (K=4, 9 variables) | 5,4 % · 7,3 % · **28,2 %** · **31,2 %** (contra 17,0 % general) | validación externa, empalma con el supervisado |
| Antigüedad media del cluster con más capacitación | 0,99 años | los Nuevos se reconocen sin etiqueta |
| Ward cortado en 2, contra enganchados/no enganchados | ARI **0,993** | el dendrograma: el primer corte es el enganche |
| Single linkage cortado en 2 | 599 + 1 | el encadenamiento, para comparar linkages |
| Perfil × `area` | Cramér's V = 0,081 | el área (el "género musical" que asigna RRHH) no captura los perfiles |

> **Aviso para quien arme la clase:** en las 9 variables, el silhouette de K=3 y el de K=4
> quedan casi empatados (0,367 contra 0,370). El codo de W sí marca K=4 claramente. No es un
> defecto: elegir K es ambiguo en datos realistas (ISLP §12.4.3), pero conviene saberlo antes
> de afirmar en una slide que "el silhouette elige 4".

> **Nota de entorno (05/10/2026):** el generador necesita `scikit-learn` y `statsmodels`
> (`requirements.txt`). Correrlo con el `.venv` del repo.


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
| Se usa en | **Ninguna clase publicada.** Se usaba en la vieja clase de PCA, archivada el 28/09/2026 en `clases/_archivo/clase-10-pca-uci/`; la Clase 9 (PCA) usa ahora `nimbus_desempeno.csv` |

**Uso pedagógico en la vieja Clase 10 (archivada).** Es la base de datos que acompaña toda la explicación de
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
