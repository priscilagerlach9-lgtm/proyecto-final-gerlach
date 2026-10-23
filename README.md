# Rambla SA — Perfiles de riesgo del inversor uruguayo y carteras modelo

Proyecto Final C · Programación Analítica II · Universidad de Montevideo
Segundo semestre 2026 · Docente: Camila Palomeque
Autora: Priscila Gerlach

---

## Qué hace este proyecto

Rambla SA asignaba el perfil de riesgo de cada cliente preguntándoselo directamente
(«¿usted se considera conservador, moderado o agresivo?») y armaba la cartera con esa
respuesta. El problema es que la gente se describe como quiere que la vean, no como se
comporta.

Este proyecto reemplaza esa autodeclaración por una segmentación construida con datos de
comportamiento. Concretamente:

1. **Limpia y audita** la encuesta anual de perfil y el registro de operaciones de la firma
   (duplicados, centinelas de faltante, categorías mal escritas, montos en tres monedas).
2. **Reconstruye la cartera efectiva** de cada cliente a partir de lo que realmente operó, y
   le calcula un **riesgo efectivo** ponderado por monto invertido.
3. **Cuantifica la brecha** entre el perfil declarado y el perfil que revela la cartera, y
   separa los dos desajustes: el conservador con cartera agresiva (riesgo legal) y el
   agresivo con cartera conservadora (oportunidad comercial perdida).
4. **Contrasta las expectativas de retorno** contra lo que rinden los instrumentos que el
   cliente tiene, en términos reales (descontando inflación y tipo de cambio).
5. **Propone perfiles nuevos con carteras modelo** del catálogo de la firma, respetando
   mínimos de inversión y necesidad de liquidez.

Fecha de corte de todos los datos: **30 de junio de 2026**.

---

## Estructura del repositorio

```
proyecto-final-gerlach/
├── README.md               Este archivo
├── requirements.txt        Dependencias con versión fijada
├── .gitignore
├── datos/                  Datos de entrada (no se modifican nunca)
│   ├── encuesta_inversores.csv
│   ├── movimientos_cartera.csv
│   ├── catalogo_instrumentos.csv
│   ├── tasas_referencia.csv
│   └── Info.txt            Diccionario de datos original de la cátedra
├── notebooks/
│   └── proyecto_final_rambla.ipynb    Todo el análisis, en orden
└── salidas/                Tablas y gráficos que genera el notebook
```

Los archivos de `datos/` son de **solo lectura**: el notebook los carga y trabaja sobre
copias en memoria. Todo lo que hay en `salidas/` es regenerable corriendo el notebook.

---

## Cómo crear el entorno

Requiere **Python 3.11 o superior** (desarrollado y probado con 3.14.5).

### Windows (PowerShell)

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### macOS / Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

---

## Cómo correrlo

```bash
jupyter lab notebooks/proyecto_final_rambla.ipynb
```

Y en el menú: **Run → Restart Kernel and Run All Cells**.

El notebook está pensado para correr **de arriba abajo sin errores y sin intervención**.
Usa rutas relativas al propio notebook, así que funciona desde cualquier carpeta siempre
que se mantenga la estructura del repositorio.

Para verificarlo sin abrir JupyterLab:

```bash
jupyter execute notebooks/proyecto_final_rambla.ipynb
```

---

## En qué orden está el notebook

| Sección | Contenido |
|---|---|
| **0** | Configuración, rutas y constantes de conversión |
| **1** | Análisis exploratorio y correcciones de calidad de datos |
| **2** | Creación de variables: monedas, cartera efectiva, riesgo efectivo |
| **3** | Análisis de negocio: la brecha, las expectativas, qué explica el comportamiento |
| **4** | Visualización (seis gráficos, uno interactivo con Plotly) |
| **5** | Perfiles nuevos y carteras modelo |

Cada sección arranca con una celda Markdown que dice qué se va a hacer y termina con las
conclusiones del tramo. **Toda decisión de limpieza (borrar, imputar, corregir) va
documentada con el porcentaje de la base afectado antes y después**, según la regla de la
Etapa 1 de la consigna, y queda registrada automáticamente en
`salidas/1_bitacora_decisiones.csv`.

---

## Resultados principales

### La brecha entre lo que dicen y lo que hacen

**El 50,2 % de los clientes con cartera tiene un perfil de riesgo real distinto del que
declaró.** Pero el número que importa es el **kappa ponderado de Cohen: 0,25** (0,18 sin
ponderar). Corrigiendo por el acuerdo esperado por azar, la autodeclaración del cliente
tiene un acuerdo *débil* —insignificante, sin ponderar— con su comportamiento real.

El hallazgo es robusto: probando **120 combinaciones de cortes** distintas, el desajuste se
mantiene entre 45,4 % y 57,5 % y el kappa nunca alcanza 0,40.

| Severidad del desajuste | Clientes | Patrimonio |
|---|---|---|
| Se dijo conservador y tiene cartera agresiva (**riesgo legal**) | 22 | USD 435.579 |
| Cartera más riesgosa que lo declarado, en total | 138 (22,0 %) | USD 3.064.919 |
| Cartera más conservadora (**oportunidad comercial**) | 177 (28,2 %) | USD 3.653.923 |

La asimetría es jurídica: según la normativa del BCU, apartarse del perfil asignado y
consentido por el cliente genera responsabilidad del intermediario.

### Las expectativas no son alcanzables

Los **113 clientes que esperan más de 30 % anual** obtienen 7,8 % nominal y 4,7 % real.
**Ninguno** tiene una cartera capaz de rendir lo que espera, y ninguna cartera posible con
este catálogo podría: el instrumento más rentable rinde 21,5 % real. Además, del tramo de
expectativa más bajo al más alto la volatilidad **se triplica** para ganar apenas un cuarto
más de retorno.

### Qué explica de verdad el comportamiento

La relación aparente «el que sabe más de finanzas invierte más conservador» (coeficiente
−0,219, p < 0,0001) **se desarma al controlar por fuente de información y expectativa de
retorno**: pierde el 71 % de su magnitud y deja de ser significativa (p = 0,15). La cadena
real es que quien sabe menos se informa por redes sociales, quien se informa por redes
espera retornos más altos, y quien espera más toma más riesgo. **De dónde saca la
información el cliente pesa 1,8 veces más que cuánto sabe.**

### Los cuatro perfiles nuevos

La segmentación se construyó con k-medias sobre diez variables de comportamiento y situación
financiera, **sin usar la autodeclaración ni la edad**. Las siluetas son bajas (0,22–0,25),
así que la elección de cuatro grupos se justifica por interpretabilidad y se valida contra
variables que no participaron del agrupamiento: edad (F = 6,97, p = 0,0001), puntaje del test
(F = 18,40, p < 0,00001) e ingreso (F = 3,36, p = 0,019) difieren significativamente entre
perfiles.

| Perfil | Clientes | Patrimonio | Riesgo | Qué lo define |
|---|---|---|---|---|
| **El Previsor** | 231 (37,6 %) | USD 4,71 M (39,4 %) | 1,97 | **Los 231 tienen fondo de emergencia** (6,5 meses). Mayor edad, mejor test, mayor ingreso |
| **El Desprotegido** | 215 (35,0 %) | USD 3,92 M (32,8 %) | 2,31 | Invierte igual de prudente pero **ninguno tiene fondo de emergencia** |
| **El Optimista** | 154 (25,0 %) | USD 3,13 M (26,2 %) | 3,19 | **84 % se informa por redes sociales**, espera 23,2 % anual, peor test (3,58) |
| **El Apostador** | 15 (2,4 %) | USD 0,19 M (1,6 %) | 4,60 | **83 % de su cartera en cripto**, volatilidad 55 %, el de menor patrimonio |

**Los cuatro perfiles se superponen casi por completo en patrimonio** y se separan por
comportamiento. Segmentar por monto —como hizo la industria durante décadas— los habría
mezclado en un solo grupo.

El contraste que más importa es entre los dos primeros: **invierten casi igual y tienen
patrimonios equivalentes, pero uno cubre 6,5 meses de gastos y el otro cero.** Esa diferencia
no aparece en ninguna escala de riesgo y cambia por completo qué se les puede recomendar.

### Las carteras modelo

Cada cartera verifica **en código** sus tres restricciones: los pesos suman 100 %, cada
posición supera el mínimo de inversión del instrumento para la cartera mediana del perfil, y
la liquidez se ajusta a si el perfil tiene colchón. Un `assert` corta la corrida si alguna
falla.

| Perfil | Riesgo | Volatilidad | Retorno real | Líquido | Patrimonio mínimo |
|---|---|---|---|---|---|
| El Previsor | 2,10 | 5,8 % | 3,56 % | 75 % | USD 4.673 |
| El Desprotegido | 1,70 | 3,8 % | 2,84 % | 65 % | USD 6.667 |
| El Optimista | 3,10 | 13,2 % | 5,09 % | 90 % | USD 10.000 |
| El Apostador | 2,35 | 11,6 % | 5,12 % | 80 % | USD 4.673 |

Se incluye además una **variante ampliada** por perfil, que incorpora los instrumentos de
mínimo alto y sólo es viable por encima del patrimonio que el notebook calcula.

### El hueco del catálogo

Con una cartera típica de unos USD 11.700, los instrumentos accesibles sin concentrar más de
un cuarto del patrimonio son los de **riesgo 1 y 2** o los de **riesgo 4 y 5**. El tramo
intermedio está fuera de alcance: las Notas del Tesoro en UI exigen el 39 % de esa cartera,
las obligaciones negociables el 43 %, el fideicomiso el 52 % y el inmueble de renta el 683 %.
El único instrumento de riesgo 3 accesible es el ahorro voluntario en AFAP, que **pierde
poder de compra** (−0,47 % real) y tiene liquidez muy baja.

Las carteras no saltan de riesgo 1-2 a riesgo 4-5 porque los clientes elijan así: **saltan
porque el menú no ofrece otra cosa.** Es una limitación de producto que ningún rediseño del
formulario resuelve.

### Otros hallazgos

- **Los faltantes no son aleatorios.** Quien no declara su patrimonio tiene una cartera real
  un 65 % más grande (en mediana) que quien sí lo declara. Borrar las filas incompletas
  habría eliminado del análisis al segmento más valioso de la firma.
- **La escala de riesgo de la firma está comprimida en el tope.** Correlaciona 0,975 con el
  estándar regulatorio SRRI, pero le asigna el mismo nivel 5 a las acciones (24 % de
  volatilidad) y a las criptomonedas (65 %). Hay 19 clientes que el estándar clasifica como
  agresivos y la escala interna no.
- **El perfil más ilíquido es el moderado, no el agresivo**: tiene el 54,4 % de su cartera
  en instrumentos de liquidez muy baja, contra 12,4 % del agresivo.
- **El 32,3 % de los clientes** tiene al menos una respuesta incompatible con otra del mismo
  formulario.

---

## Decisiones metodológicas y su fundamento

Las decisiones de criterio no se tomaron por gusto: se anclaron en estándares citables.

| Decisión | Fundamento |
|---|---|
| Cortes del riesgo en **2,5 / 3,5** sobre la escala 1-5 | Reproducen con 93 % de acuerdo la partición del **SRRI** (guía CESR/ESMA 10-673), que clasifica productos en siete clases por volatilidad anualizada |
| El efectivo se **parte en dos**: hasta 3 meses de ingreso es reserva (fuera de la cartera), el resto es asignación (dentro) | La práctica estándar de planificación financiera deja la reserva de emergencia fuera de la cartera de inversión porque cumple otra función |
| Riesgo **ponderado por monto**, no promedio simple | 100 clientes (15,9 %) difieren en más de 0,5 puntos entre ambas medidas |
| El patrimonio faltante se imputa con la **cartera observada** | Los faltantes son MNAR; el registro de operaciones es una fuente independiente que el cliente no puede falsear |
| Fechas parseadas **por patrón**, no con `format="mixed"` | El archivo mezcla dos formatos y `mixed` con `dayfirst` fechaba 101 encuestas después de la fecha de corte |
| Retorno real deflactado por la inflación de **cada moneda** | Un retorno nominal en pesos no se compara con uno en dólares; se incluye una versión de sensibilidad en pesos reales |

---

## Qué hay en `salidas/`

Todo lo de esta carpeta lo regenera el notebook al correr.

| Archivo | Contenido |
|---|---|
| `1_bitacora_decisiones.csv` | Las 17 decisiones de limpieza con el % de la base afectado |
| `1_4_faltantes_por_columna.png` · `1_7_faltante_no_aleatorio.png` | Gráficos de la auditoría de calidad |
| `3_clientes_criticos_sobreexpuestos.csv` | Los 22 clientes del caso crítico, para acción comercial |
| `3_inconsistencias_por_cliente.csv` | Las 10 reglas de inconsistencia, cliente por cliente |
| `4_1_radar_perfiles.html` | **Gráfico interactivo** (Plotly): abrir en el navegador |
| `4_1` a `4_6` `.png` | Los seis gráficos de la Etapa 4 en estático |
| `5_1_radar_perfiles_nuevos.html` | **Gráfico interactivo** de los cuatro perfiles nuevos |
| `5_1_radar_perfiles_nuevos.png` | El mismo radar en estático |

---

## Calidad del código

El código del notebook pasa por el flujo de tres pasos del curso. La configuración está en
`setup.cfg`, así que las herramientas usan el mismo criterio para cualquiera que clone el
proyecto, sin pasar parámetros a mano.

```bash
# 1 · formatear
python -m black --line-length 88 notebooks/

# 2 · estilo y errores (lee setup.cfg)
python -m flake8 notebooks/

# 3 · diseño
python -m pylint notebooks/
```

| Herramienta | Estado |
|---|---|
| **black** | Las 97 celdas de código salen formateadas a 88 columnas |
| **flake8** | Sin avisos de pyflakes, sin imports mal ubicados, sin continuaciones mal indentadas |
| **pylint** | **9,54 / 10** |

Las exclusiones de pylint están justificadas una por una en `setup.cfg`: `dict(...)` anidado
en las llamadas a Plotly y Matplotlib, que es el estilo de la documentación de esas
librerías; nombres de parámetro que coinciden con variables del notebook, inevitable cuando
todo vive en el ámbito global; y dos falsos positivos. El criterio es leer la salida aviso
por aviso en vez de perseguir el puntaje.

Los gráficos usan **Matplotlib y Seaborn**, con la interfaz `fig, ax`, y **Plotly** para los
dos gráficos interactivos. Ninguno tiene el eje truncado.

---

## Limitaciones declaradas

- **La volatilidad de las carteras modelo está sobreestimada.** Se calcula como promedio
  ponderado de las volatilidades individuales, lo que equivale a suponer que todos los
  activos se mueven juntos. Es una cota superior; la volatilidad real sería menor. Calcularla
  exigiría una matriz de covarianzas que el catálogo no trae.
- **La cartera es costo de adquisición, no valor de mercado.** El registro de operaciones no
  tiene ventas ni rescates, así que se asume que ningún cliente desinvirtió.
- **No se puede afirmar causalidad.** Los datos son de corte transversal: que la fuente de
  información prediga el riesgo no prueba que lo cause.
- **Los 73 clientes con operaciones pero sin encuesta** quedaron fuera del análisis. Son el
  1,7 % del monto y sus identificadores pertenecen a otro bloque de numeración.

---

## Nota sobre los gráficos

Los gráficos interactivos están hechos con **Plotly**. Para que se vean en GitHub (que no
ejecuta JavaScript al previsualizar un `.ipynb`), cada gráfico interactivo se acompaña de
una versión estática en `salidas/`. Si al abrir el notebook en GitHub un gráfico de Plotly
aparece en blanco, es esa limitación del visor: corriendo el notebook localmente se ve.

---

## Fuente de los datos

Datos sintéticos provistos por la cátedra, generados para uso docente e inspirados en
[kaggle.com/datasets/nitindatta/finance-data](https://www.kaggle.com/datasets/nitindatta/finance-data),
adaptados al mercado uruguayo. **Rambla SA es una firma ficticia** y los rendimientos del
catálogo son ilustrativos: nada de lo que hay acá constituye asesoramiento de inversión.
