# 🛒 Segmentación de clientes de un e-commerce con K-Means y RFM

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/bentadev/segmentacion-clientes-kmeans/blob/main/notebooks/segmentacion_clientes_kmeans_rfm.ipynb)

¿Todos los clientes de una tienda online compran igual? En este proyecto analizamos **536.350 registros de ventas** para descubrir qué tipos de clientes hay. Para eso resumimos a cada cliente con tres números (**RFM**) y los agrupamos con **K-Means**. El resultado: **4 segmentos** de clientes y una acción comercial para cada uno.

---

## 📌 Contexto académico

| | |
|---|---|
| 🏛️ **Institución** | IFTS N°33 |
| 🎓 **Carrera** | Ciencia de Datos e IA |
| 📚 **Materia** | Modelizado de Minería de Datos |
| 📝 **Trabajo** | TP N°5 (grupal): aplicar clustering (K-Means o DBSCAN) o reglas de asociación (Apriori) a un dataset |

El objetivo del trabajo es integrar y consolidar lo aprendido en clase mediante la práctica directa. Nuestro grupo eligió **K-Means**.

## 🎯 Objetivo

Segmentar a los clientes según su comportamiento de compra para saber **quiénes son, cuánto aportan y qué conviene hacer con cada grupo**.

---

## 📦 Dataset

Registro de transacciones de una tienda online entre **diciembre de 2018 y diciembre de 2019**. Cada fila es un producto dentro de una compra.

| Columna | Qué representa |
|---|---|
| `TransactionNo` | Número de compra (si empieza con `C`, es una cancelación) |
| `Date` | Fecha de la compra |
| `ProductNo` / `ProductName` | Código y nombre del producto |
| `Price` | Precio unitario |
| `Quantity` | Cantidad comprada |
| `CustomerNo` | Número de cliente |
| `Country` | País del cliente |

- 📏 **Tamaño original:** 536.350 filas y 8 columnas (~43 MB; en el repositorio está comprimido en `.zip`).
- 👥 **Contenido:** 4.738 clientes, 3.768 productos y 38 países; cerca del 90 % de las compras son del Reino Unido.
- 🔗 **Fuente y licencia:** descargado de Kaggle (https://www.kaggle.com/datasets/gabrielramos87/an-online-shop-business), con licencia [CC0: Public Domain](https://creativecommons.org/publicdomain/zero/1.0/), que permite copiarlo y redistribuirlo.

---

## 🧭 ¿Por qué K-Means con RFM?

**RFM** resume a cada cliente en tres números:

- ⏱️ **R (Recencia):** días desde su última compra.
- 🔁 **F (Frecuencia):** cantidad de compras distintas.
- 💰 **M (Monetario):** dinero total gastado.

**K-Means** agrupa clientes con RFM parecido en *clusters*.

- ✅ **K-Means:** hay fecha, cliente y gasto; los segmentos se explican fácil y sugieren acciones.
- ❌ **Apriori:** daría reglas muy obvias (el mismo producto en otros colores) sobre ~3.700 productos dispersos.
- ❌ **DBSCAN:** es difícil de ajustar y los clientes no forman zonas claras de densidad.

---

## 🔬 Metodología

El notebook sigue este orden, y cada paso prepara al siguiente:

**1. Carga e inspección.** Se revisa el dataset sin modificarlo y se detectan los problemas:

| Hallazgo | Por qué molesta |
|---|---|
| 5.200 filas duplicadas | Cuentan la misma compra dos veces |
| 55 filas sin cliente | Sin cliente no se puede calcular el RFM |
| 8.585 cancelaciones | Cantidad negativa y `C` en el número de compra (son las mismas filas) |
| 2 países ambiguos | `Unspecified` y `European Community` |
| Valores extremos | `Quantity` llega a ±80.995 y `Price` a 660,62 |
| `Date` como texto | Sin fecha real no hay recencia |

**2. Limpieza.** Se trabaja sobre una copia, y el resultado es **536.350 → 521.077 filas (−2,85 %)**. Para los valores extremos se cortó en el **percentil 99,9** de `Quantity` y `Price`:

| Percentil | Corte `Quantity` | Corte `Price` | Filas eliminadas |
|---|---|---|---|
| 99 % | 100 | 27,30 | 9.158 (1,75 %) |
| 99,5 % | 160 | 30,71 | 4.978 (0,95 %) |
| **99,9 % (elegido)** | **456** | **44,37** | **1.024 (0,20 %)** |
| 99,99 % | 1.437 | 594,50 | 83 (0,02 %) |

Con 99 % se pierden compras mayoristas reales; con 99,99 % quedan precios de hasta 594 (la mediana es 11,94). Es un criterio del grupo, no una regla fija.

**3. Armado del RFM.** Se pasa de "un producto por fila" a **una fila por cliente** (4.692 clientes).

**4. Preparación de los datos.** El gasto y la frecuencia tienen colas larguísimas, así que se aplica **logaritmo** y luego **escalado** (promedio 0 y desvío 1):

| Variable | Asimetría original | Con logaritmo |
|---|---|---|
| Recencia | 1,2 | −0,4 |
| Frecuencia | 12,4 | 1,2 |
| Monetario | 24,5 | −0,6 |

**5. Elección de k.** Se probó de 2 a 6 grupos:

| k | Inercia | Silueta |
|---|---|---|
| 2 | 7.509 | **0,402** |
| 3 | 5.719 | 0,311 |
| **4** | **4.663** | **0,316** |
| 5 | 3.896 | 0,294 |
| 6 | 3.340 | 0,310 |

La silueta más alta es k = 2, pero solo separaría "buenos" y "malos" clientes. Entre k = 3 y 6, **k = 4** tiene la mejor silueta y da cuatro perfiles que se pueden explicar y accionar. El notebook suma además Davies-Bouldin, Calinski-Harabasz y estabilidad entre semillas (ARI).

**6. Modelo final e interpretación.** K-Means con `k = 4` y `random_state = 42`, y perfil de cada grupo.

---

## 📊 Resultados

| Segmento | Clientes | % Clientes | % Ingresos | Recencia | Frecuencia | Monetario | Acción sugerida |
|---|---|---|---|---|---|---|---|
| 🏆 **VIP / Campeones** | 950 | 20,2 % | **64,7 %** | ~16 días | ~12 compras | ~38.150 | Fidelizar: beneficios y acceso anticipado |
| 🔄 **En Riesgo** | 1.407 | 30,0 % | 25,1 % | ~99 días | ~3 compras | ~10.000 | Reactivar: ofertas personalizadas |
| 🌱 **Prometedores** | 956 | 20,4 % | 6,2 % | ~20 días | ~2 compras | ~3.600 | Hacer crecer: venta cruzada y segunda compra |
| 💤 **Inactivos** | 1.379 | 29,4 % | 4,0 % | ~204 días | ~1 compra | ~1.630 | Reconquista de bajo costo |

Los valores de Recencia, Frecuencia y Monetario son **promedios** de cada grupo.

**Lo más importante:**

- 📌 **Concentración:** el 20 % de los clientes genera cerca del **65 %** de los ingresos.
- ⚠️ **Fuga:** casi el 30 % compró una vez y lleva más de 6 meses sin volver; aporta solo el 4 %.
- 🌱 **Oportunidad:** los clientes prometedores pueden convertirse en fieles con un buen estímulo.

---

## ⚠️ Limitaciones

- 🎚️ **Criterio en k:** la silueta más alta es k = 2; se eligió k = 4 por utilidad.
- 🇬🇧 **Un mercado dominante:** ~90 % de las compras son del Reino Unido.
- 📅 **Un solo año:** la recencia depende de ese corte.
- 💰 **Promedios inflados:** unos pocos compradores mayoristas elevan el promedio de gasto.
- 💱 **Moneda:** el dataset no indica la moneda de `Price`.
- 🧪 **Semilla fija:** `random_state=42` permite repetir los resultados; otra semilla puede cambiarlos levemente.

---

## ▶️ Cómo ejecutarlo

**Opción 1: Google Colab (recomendada)**

1. Hacé clic en el botón **Abrir en Colab** de arriba.
2. Ejecutá todo: *Entorno de ejecución → Ejecutar todo*. El notebook descarga los datos directamente desde este repositorio.

**Opción 2: en tu computadora**

```bash
# 1. Clonar el repositorio
git clone https://github.com/bentadev/segmentacion-clientes-kmeans.git
cd segmentacion-clientes-kmeans

# 2. Instalar las librerías
pip install -r requirements.txt

# 3. Abrir el notebook (el dataset ya está en data/)
cd notebooks
jupyter notebook segmentacion_clientes_kmeans_rfm.ipynb
```

En la celda de carga, comentá la línea de la URL y descomentá la de la ruta local (`../data/comercio_electronico.zip`). Pandas lee el `.zip` directamente, sin descomprimirlo.

> 💡 **Los gráficos 3D son interactivos** (se rotan y se hace zoom) solo al ejecutar el notebook en Colab o Jupyter. GitHub muestra el notebook sin esa interactividad.

---

## 🗂️ Estructura del repositorio

```
├── README.md                   ← este archivo
├── requirements.txt            ← librerías necesarias
├── notebooks/
│   └── segmentacion_clientes_kmeans_rfm.ipynb
├── data/
│   ├── README.md               ← descripción del dataset
│   └── comercio_electronico.zip ← datos (CSV comprimido)
```

## 🛠️ Tecnologías

Python · pandas · NumPy · Matplotlib · Plotly · scikit-learn

## 👥 Integrantes

- _Martin: @bentadev_
- _Maria: @yanin-dev_
- _Rosmery: @ros-arango_
- _Emmanuel: @Emmaflores94_
- _Priscila: @priscila-chino_
