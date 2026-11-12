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
comportamiento y situación financiera. Concretamente:

1. **Limpia y audita** la encuesta anual de perfil y el registro de operaciones de la firma
   (duplicados, centinelas de faltante, categorías mal escritas, montos en tres monedas).
2. **Reconstruye la cartera efectiva** de cada cliente a partir de lo que realmente operó, y
   le calcula un **riesgo efectivo** ponderado por monto invertido.
3. **Cuantifica la brecha** entre el perfil declarado y el que revela la cartera, y separa
   los dos desajustes: el conservador con cartera agresiva (riesgo legal) y el agresivo con
   cartera conservadora (oportunidad comercial).
4. **Contrasta las expectativas de retorno** contra lo que rinden los instrumentos que el
   cliente tiene, en términos reales.
5. **Propone perfiles nuevos, una regla para asignarlos a un cliente nuevo y carteras
   modelo** del catálogo de la firma, respetando mínimos de inversión y liquidez.

Fecha de corte de todos los datos: **30 de junio de 2026**.

---

## Estructura del repositorio

```
proyecto-final-gerlach/
├── README.md               Este archivo
├── requirements.txt        Dependencias con versión fijada
├── .gitignore
├── setup.cfg               Configuración de flake8 y pylint
├── datos/                  Datos de entrada (no se modifican nunca)
│   ├── encuesta_inversores.csv
│   ├── movimientos_cartera.csv
│   ├── catalogo_instrumentos.csv
│   ├── tasas_referencia.csv
│   └── Info.txt            Diccionario de datos original de la cátedra
├── notebooks/
│   └── proyecto_final_rambla.ipynb    Todo el análisis, en orden
└── salidas/                Tablas que genera el notebook
```

Los archivos de `datos/` son de **solo lectura**: toda la limpieza se hace con código, sobre
copias en memoria. Todo lo que hay en `salidas/` es regenerable corriendo el notebook.

---

## Cómo crear el entorno

Requiere **Python 3.11 o superior** (desarrollado y probado con 3.14).

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

Si se abre el notebook en VS Code, hay que elegir como kernel el Python de `.venv`.

---

## Cómo correrlo

```bash
jupyter lab notebooks/proyecto_final_rambla.ipynb
```

Y en el menú: **Run → Restart Kernel and Run All Cells**. El notebook corre **de arriba
abajo sin errores y sin intervención**, y encuentra los datos solo, desde `notebooks/` o
desde la raíz del repositorio.

Para verificarlo sin abrir JupyterLab:

```bash
jupyter execute notebooks/proyecto_final_rambla.ipynb
```

---

## En qué orden está el notebook

1. **Portada y supuestos**: el caso, la fecha de corte que fija la consigna y los supuestos
   que afectan los resultados.
2. **Setup**: librerías, rutas y estilo de los gráficos.
3. **Exploración y calidad de datos**: se cargan los datos crudos y cada problema se
   muestra, se decide y se corrige en el momento en que aparece, nunca antes.
4. **Creación de variables**: monedas, cartera efectiva, riesgo efectivo, cortes, retorno
   real.
5. **Análisis de negocio**: la brecha, las expectativas, qué explica el comportamiento.
6. **Visualización** con Plotly.
7. **Perfiles y carteras modelo**: segmentación, regla de asignación, carteras modelo y
   rediseño del formulario.
8. **Conclusiones**.

Cada parámetro (meses de reserva, cortes, límites de las carteras) se define justo antes
de usarse, con una celda que explica de dónde sale. Toda decisión de limpieza (borrar,
imputar, corregir) imprime cuántas filas toca.

---

## Resultados principales

### La brecha entre lo que dicen y lo que hacen

**El 50,2 % de los clientes con cartera tiene un perfil de riesgo real distinto del que
declaró.** El **kappa ponderado de Cohen es 0,25** (0,18 sin ponderar): corrigiendo por el
acuerdo esperado por azar, la autodeclaración tiene un acuerdo *débil* con el
comportamiento real. Con **120 combinaciones de cortes** distintas el desajuste queda entre
45 % y 58 % y el kappa nunca llega a 0,40.

| Desajuste | Clientes | Patrimonio |
|---|---|---|
| Se dijo conservador y tiene cartera agresiva (**riesgo legal crítico**) | 22 | USD 435.579 |
| Cartera más riesgosa que lo declarado, en total | 138 (22,0 %) | USD 3.064.919 |
| Cartera más conservadora (**oportunidad comercial**) | 177 (28,2 %) | USD 3.653.923 |

Los sobreexpuestos se informan más por redes (38 % contra 25 % de los que coinciden) y
esperan más de 30 % anual casi el doble de veces (27 % contra 15 %). La cantidad de
sobreexpuestos depende de los cortes (de 8 % a 39 %); el desajuste total, no.

### Las expectativas no son alcanzables

Los **113 clientes que esperan más de 30 % anual** obtienen 11,3 % nominal en pesos y 4,7 %
real. Sólo 5 llegan a 30 % nominal, con carteras casi todas en cripto, y en términos reales
ninguno: ni la cripto (21,5 % real) llega a 30 % en valor esperado.

### Qué explica de verdad el comportamiento

La relación «el que sabe más de finanzas invierte más conservador» (coeficiente −0,22,
p < 0,0001) **se desarma al controlar por fuente de información y expectativa de retorno**:
baja a −0,06 y deja de ser significativa (p = 0,14). Es mediación: quien sabe menos se
informa por redes, quien se informa por redes espera más, y quien espera más toma más
riesgo. La parte del efecto que pasa por ese camino es −0,16, con un intervalo del 95 % de
−0,21 a −0,11. La regresión usa errores estándar robustos y las correlaciones son de
Spearman, con corrección de Holm por comparaciones múltiples.

### Los cuatro perfiles nuevos

Construidos con k-medias sobre siete variables de comportamiento y situación financiera,
**sin la autodeclaración**. El codo es suave y la silueta baja (0,23 a 0,27): cuatro grupos
es una decisión de negocio, porque es la menor cantidad que separa a quien tiene fondo de
emergencia de quien no. Los perfiles se distinguen por tres señales: más de la mitad en
cripto, se informa por redes, no tiene fondo. En 250 remuestreos el índice de Rand mediano
es 0,97 (percentil 10: 0,36, porque el Apostador tiene sólo 15 clientes).

| Perfil | Clientes | Cartera en la firma | Riesgo | Qué lo define |
|---|---|---|---|---|
| **El Previsor** | 220 (35,0 %) | USD 4,56 M (37,4 %) | 1,92 | Todos tienen fondo de emergencia (6,4 meses). Mayor edad, mejor test, mayor ingreso |
| **El Desprotegido** | 226 (36,0 %) | USD 4,06 M (33,2 %) | 2,40 | Invierte con algo más de riesgo que el Previsor pero **ninguno declara fondo de emergencia** |
| **El Optimista** | 167 (26,6 %) | USD 3,40 M (27,9 %) | 3,04 | El 96 % se informa por redes; espera más de 20 % anual; peor test |
| **El Apostador** | 15 (2,4 %) | USD 0,19 M (1,5 %) | 4,60 | El 83 % de su cartera en cripto, volatilidad 55 %, el de menor patrimonio |

**Para un cliente nuevo**, dos preguntas del formulario (fuente de información y meses de
gastos cubiertos) asignan bien el perfil en el 94,8 % de los casos de prueba, contra el 37 %
de asignar a todos el más frecuente; promediando lo que reconoce de cada perfil, el acierto
es 72,7 %, porque el Apostador no se detecta con ninguna respuesta. A ese perfil lo detecta el
control anual de la cartera.

### Las carteras modelo

Para los perfiles sin fondo de emergencia, primero se aparta la reserva de tres meses de
ingreso (a 70 clientes la reserva se les lleva toda la cartera) y la cartera se arma con
lo que queda. Cuando lo que el cliente quiere supera lo que puede perder, manda la
capacidad: sin fondo de emergencia, la volatilidad no pasa de 10 %. Cada cartera verifica
**en código** que los pesos sumen 100 %, que ninguna posición pase de 30 %, que cada una
supere el mínimo de inversión para el monto invertible mediano del perfil, que la liquidez
sea acorde y que se respete la capacidad. Un `assert` corta la corrida si alguna falla.

- **El Previsor** (invertible mediano USD 11.183): riesgo de 1,92 a 1,80, volatilidad 4,3 %,
  retorno real de 2,57 % a 3,09 %.
- **El Desprotegido** (USD 6.765): primero el colchón; riesgo de 2,40 a 1,85, volatilidad de
  5,8 % a 4,5 %, retorno real de 2,92 % a 3,02 %.
- **El Optimista** (USD 11.023): riesgo de 3,04 a 2,27, volatilidad de 10,1 % a 7,9 %, retorno
  real de 3,90 % a 4,02 %. 53 Optimistas consintieron el perfil conservador y necesitan
  re-perfilarse.
- **El Apostador** (USD 1.801): 30 % fondo local, 30 % plazo fijo en dólares, 20 % dólares
  líquidos, 10 % cripto y 10 % plazo fijo en pesos. La volatilidad baja de 55 % a 7,9 %. El
  retorno promedio engaña con tanta volatilidad: hoy promedia 18,4 % real pero crece 7,4 %
  compuesto; la propuesta crece 2,3 %, y si la cripto cae 60 % pierde el 6 % en vez del
  49,6 %.

La volatilidad es un techo en la moneda de cada instrumento y no incluye riesgo cambiario.

### El formulario

Deciden la cartera la tolerancia a la pérdida máxima (separa 1,90 puntos de riesgo), la
frecuencia de monitoreo (1,61), la fuente de información (1,07), el retorno esperado (1,04) y
el fondo de emergencia. La reacción ante una caída separa 1,36 pero al revés (los que
venderían todo tienen las carteras más riesgosas), así que queda como pregunta de control.
Las demográficas no pasan de 0,32 y se mantienen sólo porque las exige la norma. Las siete
preguntas de preferencia entre instrumentos se sacan: ninguna anticipa lo que el cliente
compra una vez corregido por comparaciones múltiples.

### Otros hallazgos

- **Los faltantes no son aleatorios.** Quien no declara su patrimonio tiene una cartera real
  un 76 % más grande en mediana (Mann-Whitney, p = 0,001). Se imputa con lo operado por la
  relación declarado/operado de su cuartil (de 1,78 a 1,29).
- **La escala de riesgo de la firma está comprimida en el tope.** Correlaciona 0,975 con el
  SRRI, pero le da el mismo 5 a las acciones (24 % de volatilidad) y a la cripto (65 %).
- **El perfil más ilíquido es el moderado**: el 54,4 % de su cartera en instrumentos de
  liquidez muy baja.
- **Hueco en el catálogo**: con una cartera típica de USD 11.660, el único riesgo 3 accesible
  es el ahorro en AFAP, que pierde poder de compra.

---

## Supuestos que afectan los resultados

La lista completa está al principio del notebook. Los que más pesan:

- La fecha de corte de toda la información es el 30/06/2026, la que fija la consigna.
- Todas las operaciones se pasan a dólares con el tipo de cambio y la UI del 30/06/2026.
- La cartera es costo de adquisición: no hay ventas, así que se asume que nadie desinvirtió.
- El retorno de cada instrumento es su histórico del catálogo, y el real se calcula con la
  inflación de los últimos 12 meses.
- La reserva de emergencia es 3 meses del ingreso declarado, el mínimo de la regla habitual
  de 3 a 6 meses de gastos.
- La volatilidad de las carteras modelo es un techo en la moneda de cada instrumento: no hay
  covarianzas y no incluye el riesgo cambiario.
- Los datos son de corte transversal: se habla de asociación, no de causalidad.

---

## Qué hay en `salidas/`

| Archivo | Contenido |
|---|---|
| `3_clientes_criticos_sobreexpuestos.csv` | Los 22 clientes del caso crítico, para acción comercial |
| `3_inconsistencias_por_cliente.csv` | Las reglas de inconsistencia, cliente por cliente |
| `*.parquet` | Bases limpias intermedias (no se versionan) |

---

## Calidad del código

El código sigue el flujo del curso, con la configuración en `setup.cfg`. flake8 y pylint no
leen `.ipynb`, así que se corren sobre el notebook exportado a `.py`. Las herramientas no
están en `requirements.txt` porque no hacen falta para correr el análisis:

```bash
python -m pip install black flake8 pylint
python -m jupyter nbconvert --to script notebooks/proyecto_final_rambla.ipynb --output-dir revision
python -m flake8 revision/proyecto_final_rambla.py
python -m pylint revision/proyecto_final_rambla.py
```

El código de las celdas está formateado con black a 88 columnas y flake8 no da avisos. Las
exclusiones de pylint están justificadas una por una en `setup.cfg`.

**Sobre lo que está en el repositorio.** Los datos de `datos/` están versionados, a
diferencia del `.gitignore` del curso, porque el proyecto se corrige ejecutándolo y tiene que
correr con sólo clonarlo. El notebook se sube ejecutado, con sus salidas, para que se pueda
leer sin correrlo.

Los gráficos están hechos con **Plotly**. GitHub no ejecuta JavaScript al previsualizar un
`.ipynb`, así que ahí pueden aparecer en blanco: corriendo el notebook se ven.

---

## Fuente de los datos

Datos sintéticos provistos por la cátedra, generados para uso docente e inspirados en
[kaggle.com/datasets/nitindatta/finance-data](https://www.kaggle.com/datasets/nitindatta/finance-data),
adaptados al mercado uruguayo. **Rambla SA es una firma ficticia** y los rendimientos del
catálogo son ilustrativos: nada de lo que hay acá constituye asesoramiento de inversión.
