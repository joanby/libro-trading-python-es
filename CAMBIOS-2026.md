# Cambios de la rama `update-2026`

> Esta rama es el código del libro *Python para finanzas y trading algorítmico* (segunda edición), adaptado a las librerías de hoy
> (matplotlib 3.11.2, pandas 3.0.6, yfinance 1.7.0; octubre de 2026). La rama principal sigue exactamente como en el libro.

> **Qué está comprobado y qué no.** Se ha ejecutado cada notebook entero con las versiones de
> `requirements.txt`: **27 correctos · 0 con fallo · 0 con timeout · 19 omitidos**. Comprobado: que cada celda se ejecuta sin error y que los datos
> tienen la forma del libro (histórico completo, mismas columnas, mismo orden). **No comprobado:** la
> descarga real desde Yahoo Finance, que no era accesible desde el entorno de prueba; se usó una réplica
> de su respuesta con precios inventados. **Tus números saldrán distintos** a los del libro, porque los
> precios reales han seguido moviéndose.
> Sin ejecutar: `MT5`, `Pairs_Trading_app`, `ARIMA_app`, `MT5`, `Lin_Reg_App`, `Log_Reg_App`, `MT5`, `MT5`, `SVC_app`, `SVR_app`, `MT5`, `Tree_cla_App`, `Tree_reg_App`, `ANN_reg_app`, `MT5`, `MT5`, `RNN_reg_app`, `Full Project`, `Voting_project_app` (MetaTrader 5 solo funciona en Windows, con el terminal y una cuenta).

## Cómo usarla

- **En Google Colab (como en el libro):** abre el notebook de esta rama y, cuando el código lea un CSV,
  súbelo al panel de archivos igual que en el libro.
- **En tu ordenador:** `git clone -b update-2026 https://github.com/joanby/libro-trading-python-es`, instala `pip install -r requirements.txt`
  (versiones **fijadas**: las mismas con las que se ha comprobado, para que un cambio futuro de las
  librerías no lo vuelva a romper) y deja junto al notebook los CSV que use.

## Qué ha cambiado y por qué

### 1. Descargar precios con yfinance

Notebooks: `Backtest`, `Chapter_3_course`, `Backtest`, `Chapter_4_course`, `Chapter_5_course`, `Backtest`, `Backtest`, `Chapter_7_coursre`, `Backtest`, `Chapter_8_course`, `Backtest`, `Chapter_9_course`, `Chapter_10_course`, `Backtest`, `Chapter_11_course`, `Backtest`, `Chapter_12_course`, `Backtest`, `Chapter_13_courses`, `Backtest`, `Chapter_14_course`, `Backtest`, `Chapter_15_course`, `Backtest`.

yfinance cambió tres comportamientos por defecto de `yf.download` desde que se grabó el curso
(comprobado leyendo yfinance 0.1.70, la versión de entonces, y la 1.7.0 de hoy):

| En el libro | Hoy, si no dices nada | Qué pasa con el código del libro |
|---|---|---|
| Sin fechas, descarga **todo el histórico** | Descarga **solo el último mes** | Medias largas vacías, `.loc["2020"]` da `KeyError`, backtests de un mes |
| Con solo `end="2021-01-01"`, desde el principio hasta esa fecha | **Solo el mes anterior** a esa fecha | El mismo problema, sin ningún error |
| Columnas `Open, High, Low, Close, Adj Close, Volume` | Sin `Adj Close` (`auto_adjust=True`) | `KeyError: 'Adj Close'` y *Length mismatch* al renombrar |
| Columnas simples, en ese orden | Dos niveles (precio, ticker) y en **orden alfabético** | Aunque arregles lo anterior, al renombrar por posición `open` acabaría siendo `Adj Close` |

Cada `yf.download(...)` lleva ahora los argumentos que devuelven el comportamiento del libro:

```python
yf.download("EURUSD=X", period="max", auto_adjust=False, multi_level_index=False)
```

`period="max"` se añade siempre que la llamada no tenga fecha de inicio (`start`). Donde el código renombra las columnas por
posición (`df.columns = ["open", "high", ...]`), antes se reordenan como estaban:
`[["Open", "High", "Low", "Close", "Adj Close", "Volume"]]`.

**Si escribes el código siguiendo el libro**, añade esos argumentos en tu `yf.download`.

`yf.Ticker(...).history()` no ha cambiado (ya ajustaba precios y bajaba un mes por defecto). Y los dobles
corchetes de `df[["Close"]].rolling(15).mean()` **no son un fallo**: funcionan igual en pandas 3.

### 2. pandas 3: `fillna(method="ffill")` ya no existe

Notebooks: `Chapter_7_coursre`.

Se escribe `.ffill()` (y `.bfill()` para `method="bfill"`). Hace exactamente lo mismo.

### 3. matplotlib: el estilo `seaborn` cambió de nombre

Notebooks: `Backtest`, `Chapter_3_course`, `Backtest`, `Chapter_4_course`, `Chapter_5_course`, `Backtest`, `Chapter_06_course`, `Backtest`, `Chapter_7_coursre`, `Backtest`, `Chapter_8_course`, `Backtest`, `Chapter_9_course`, `Chapter_10_course`, `Backtest`, `Chapter_11_course`, `Backtest`, `Chapter_12_course`, `Backtest`, `Chapter_13_courses`, `Backtest`, `Chapter_14_course`, `Backtest`, `Chapter_15_course`, `Backtest`, `Full Project`.

`plt.style.use('seaborn')` → `plt.style.use('seaborn-v0_8')`. Es el mismo estilo de gráficos.

### 4. TensorFlow: Keras 2, como en el libro

Notebooks: `ANN_reg_app`, `Chapter_13_courses`, `Chapter_14_course`, `RNN_reg_app`, `Chapter_15_course`.

TensorFlow trae ahora Keras 3, que ya no guarda ni carga los pesos como en el libro: `save_weights("Weights_ANN/ANN n°15")` exige un nombre acabado en `.weights.h5`, y los pesos ya entrenados que trae el repositorio no se pueden leer. Por eso el notebook empieza con `os.environ["TF_USE_LEGACY_KERAS"] = "1"` y `requirements.txt` incluye `tf-keras`: `tensorflow.keras` vuelve a ser Keras 2 y el resto del código no cambia. **Si escribes el código siguiendo el libro**, pon esas dos líneas antes de importar TensorFlow.

### 5. Errores que el libro provoca a propósito

Notebooks: `Pairs_Trading_app`, `ARIMA_app`, `Lin_Reg_App`, `Log_Reg_App`, `SVC_app`, `SVR_app`, `Tree_cla_App`, `Tree_reg_App`, `ANN_reg_app`, `RNN_reg_app`, `Voting_project_app`.

Algunas celdas dan un error a propósito para explicar algo (por ejemplo, qué es una variable local). Siguen dándolo; solo se han marcado (`raises-exception`) para que *Ejecutar todo* no se pare ahí.

### Otros cambios (pandas 3, NumPy 2, statsmodels y un ticker que ya no existe)

- **`FB` → `META` (capítulo 4).** Facebook cotiza como `META` desde junio de 2022; con `FB`, Yahoo ya no
  devuelve datos. El histórico de `META` es el mismo.
- **`serie[0]` → `serie.iloc[0]`.** En pandas 3, `serie[0]` busca la *etiqueta* `0`, ya no la posición:
  `median[i]` (capítulo 4) y `Sharpe`, `Sortino` y el drawdown máximo de `Backtest.py` (en todos los
  capítulos que lo traen).
- **`Backtest.py`: `isinstance(dfc, pd.Series)`** en vez de comparar `str(type(dfc))` con
  `"<class 'pandas.core.series.Series'>"`, que en pandas 3 ya no se llama así.
- **Capítulo 6:** las columnas `Low_time` y `High_time` se crean con `pd.NaT` (antes `np.nan`) y `duration`
  con `pd.Timedelta(0)` (antes `0`): pandas 3 ya no cambia solo el tipo de una columna numérica al meter
  una fecha. En el Monte Carlo, `np.random.shuffle(returns)` → `returns = pd.Series(np.random.permutation(returns.values), index=returns.index)`:
  el mismo barajado, que con una Serie ya no funciona.
- **NumPy 2:** `np.NaN` → `np.nan` (capítulo 7).
- **statsmodels (capítulo 8):** el `ARIMA` antiguo (`statsmodels.tsa.arima_model`) ya no existe:
  `from statsmodels.tsa.arima.model import ARIMA`, `model.fit()` sin `disp=0` y la previsión con
  `np.asarray(forecast)[0]`. Es el mismo modelo AR(1).
