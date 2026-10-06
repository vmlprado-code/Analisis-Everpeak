# Sprint 7: Análisis Exploratorio y Limpieza de Datos con Everpeak Retail
**TripleTen | Carrera de Análisis de Datos**

---

## 📌 Descripción General

Este sprint aborda el flujo completo de análisis de datos desde la exploración inicial hasta el análisis exploratorio detallado (EDA). Se utiliza un dataset real de comercio electrónico (`everpeak_retail.csv`) que contiene transacciones de una plataforma minorista.

El objetivo es desarrollar habilidades prácticas en:
- Diagnóstico de calidad de datos
- Limpieza y estandarización automatizada mediante pipelines
- Análisis estadístico descriptivo
- Segmentación de clientes basada en reglas de negocio

---

## 🏗️ Estructura del Sprint: Dos Fases

### **FASE 1: EXPLORACIÓN, LIMPIEZA Y ESTANDARIZACIÓN (Prácticas 1-4)**
Fundamento técnico para preparar datos listos para análisis.

### **FASE 2: ANÁLISIS Y SEGMENTACIÓN (Prácticas 5-11)**
Aplicación de técnicas estadísticas para extraer insights de negocio.

---

## 📂 FASE 1: Preparación de Datos

### **Práctica 1: Identificando Variables Relevantes para el Análisis**

**Objetivo:** Realizar un Análisis Exploratorio de Datos (EDA) inicial.

**Técnicas Utilizadas:**
- `df.head()` — Inspección visual de las primeras filas
- `df.info()` — Revisión de tipos de datos y valores no nulos
- `df.isna().sum()` — Conteo de valores faltantes por columna
- `df.nunique()` — Cálculo de cardinalidad (diversidad de valores únicos)
- `pd.to_datetime()` — Estandarización de formatos de fecha

**Hallazgos Clave:**
- 5,008 registros de transacciones con 11 columnas
- Valores faltantes identificados: 
  - `order_date`: 8 valores nulos
  - `city` y `state`: 100 valores nulos cada uno
  - `customer_age`: 150 valores nulos
- 1,829 clientes únicos en 10 ciudades y 9 estados
- 4 métodos de pago identificados

**Importancia:** Establece la línea base de calidad de datos y define qué columnas necesitan tratamiento especial.

---

### **Práctica 2: Manejo de Valores Ausentes o Inválidos**

**Objetivo:** Diagnosticar patrones de missingness y evaluar estrategias de imputación.

**Conceptos Teóricos:**
- **MCAR** (Missing Completely At Random): El faltante es independiente de otras variables
- **MAR** (Missing At Random): El faltante depende de otras variables observadas
- **MNAR** (Missing Not At Random): El faltante depende del valor que falta

**Técnicas Utilizadas:**
- `.groupby().mean()` — Análisis de missingness por grupos
- `.dropna()` vs `.fillna()` — Comparación de estrategias (eliminar vs imputar)
- `df[col].median()` vs `df[col].mean()` — Mediana es más robusta ante outliers

**Caso de Estudio: Faltantes en `city`**
```python
missing_city_by_pay = df["city"].isna().groupby(df["payment_method"]).mean()*100
# Resultado: ~2% faltantes en todos los métodos de pago
# Conclusión: MCAR (no relacionado con método de pago)
```

**Estrategia Adoptada:**
- Valores faltantes en columnas categóricas → Rellenar con "unknown"
- Valores faltantes en columnas numéricas → Rellenar con mediana (más robusta)
- Valores faltantes en fechas → Eliminar registros (crítico para análisis temporal)

**Importancia:** Evita pérdida de datos innecesaria y preserva la integridad analítica.

---

### **Práctica 3: Creación de Funciones y Bucles para la Limpieza**

**Objetivo:** Automatizar procesos de limpieza mediante funciones reutilizables.

**Técnicas Utilizadas:**
- **Bucles `for`** — Procesar múltiples columnas sin código repetido
- **Funciones personalizadas** — Encapsular lógica de limpieza
- `pd.to_numeric(errors="coerce")` — Conversión robusta a tipos numéricos
- `.str.strip()` — Limpieza de espacios en blanco

**Funciones Implementadas:**

```python
def convertir_columnas_numericas(df, columnas):
    for col in columnas:
        df[col] = pd.to_numeric(df[col], errors="coerce")
    return df

def step_strip_text(df):
    columnas = ["product_category", "city", "state"]
    for col in columnas:
        df[col] = df[col].str.strip()
    return df
```

**Ventajas:**
- Escalabilidad: agregar columnas sin reescribir código
- Mantenibilidad: cambios en un solo lugar
- Trazabilidad: documentación clara del proceso

**Importancia:** Establece la base para pipelines de producción.

---

### **Práctica 4: Creación de un Pipeline de Limpieza**

**Objetivo:** Orquestar todas las funciones de limpieza en un flujo coherente.

**Arquitectura del Pipeline:**

```
1. REEMPLAZO DE SENTINELS
   ↓ (reemplazar valores inválidos: -999, 999, 0, -1 → NaN)
2. CREACIÓN DE FLAGS
   ↓ (crear columnas indicadoras de datos faltantes originales)
3. IMPUTACIÓN / ELIMINACIÓN
   ↓ (rellenar o dropear según tipo de dato)
4. OUTPUT: Dataset limpio
```

**Funciones Principales:**

```python
def clean_data(df, numeric_cols, text_cols, flags_cols, 
               median_fill_cols, fill_unknown_cols, date_drop_cols):
    
    # Paso 1: Reemplazar sentinels
    df = reemplazar_sentinels_global(df, numeric_cols, text_cols)
    
    # Paso 2: Crear flags ANTES de imputar
    df = crear_flags(df, flags_cols)
    
    # Paso 3: Imputar según diagnóstico
    df = imputar_segun_diagnostico(df, median_fill_cols, 
                                   fill_unknown_cols, date_drop_cols)
    
    return df
```

**Resultado:**
- Dataset original: 5,008 filas
- Dataset limpio: Listo para análisis sin valores problemáticos
- Archivo generado: `everpeak_clean.csv`

**Importancia:** Transforma datos crudos en activos analíticos confiables.

---

## 📊 FASE 2: Análisis Estadístico y Segmentación

### **Práctica 5: Medidas Estadísticas en Columnas Numéricas**

**Objetivo:** Resumir distribuciones de variables cuantitativas.

**Estadísticas Descriptivas Clave:**
- **Tendencia Central:** media, mediana
- **Dispersión:** desviación estándar (std), rango (min-max)
- **Posición:** cuartiles (25%, 50%, 75%)

**Herramientas:**
- `.describe()` — Resumen automático de 8 estadísticos
- `.mean()` / `.median()` — Comparación para detectar outliers
- `.std()` — Variabilidad de los datos

**Caso de Estudio: Fashion vs Sports**

| Métrica | Fashion | Sports |
|---------|---------|--------|
| Media `order_value` | $8,745 | $10,365 |
| Mediana `order_value` | $8,115 | $10,569 |
| Std `order_value` | $10,319 | $8,821 |
| Conclusión | Mayor variabilidad | Más consistente |

**Insight Crítico: Quantity**
```python
# Media: 21.88 vs Mediana: 0
# Conclusión: La mayoría compra sin cantidad (0), 
#             pero algunos outliers compran en volumen
```

**Técnica de Interpretación:**
- Si `media >> mediana` → Presencia de outliers altos
- Si `media ≈ mediana` → Distribución simétrica

**Importancia:** Detecta sesgos y anomalías que afectan decisiones comerciales.

---

### **Práctica 6: Medidas Estadísticas en Columnas Categóricas**

**Objetivo:** Entender distribuciones de variables cualitativas.

**Técnicas Utilizadas:**
- `.describe()` para categóricas — Reporta: count, unique, top (valor más frecuente), freq
- `.value_counts()` — Frecuencia absoluta
- `.value_counts(normalize=True)` — Frecuencia relativa (proporción)

**Distribución Global del Dataset:**

| Variable | Top Categoría | Proporción |
|----------|---------------|-----------|
| `product_category` | Fashion | 14.78% |
| `payment_method` | Credit Card | 54.74% |
| `city` | Houston / Seattle | 10.26% |
| `state` | California | 19.54% |

**Hallazgo Crítico:**
- Dominio de tarjetas de crédito (55% del total)
- Distribución geográfica relativamente equilibrada
- Categorías de productos sin concentración extrema

**Técnica: Comparación por Segmento**
```python
# Toys: Análisis por métodos de pago
# San Francisco lidera en compras de Toys (12.1%)
# Identifica oportunidades de targeting por ciudad-categoría
```

**Importancia:** Guía estrategias de marketing y recursos comerciales.

---

### **Prácticas 7-9: Visualización de Distribuciones e Identificación de Outliers**

**Objetivo:** Detectar valores atípicos mediante reglas estadísticas (no detalladas en el contenido pero referenciadas).

**Métodos de Detección de Outliers:**
- **Regla 1.5×IQR:** Si `valor < Q1 - 1.5×IQR` o `valor > Q3 + 1.5×IQR` → Outlier
- **Desviación Estándar:** Si `|valor - media| > 3×std` → Outlier extremo
- **Visualización:** Histogramas y boxplots revelan colas y concentraciones

**Tipos de Distribuciones:**
- **Simétrica:** Media ≈ Mediana (distribución normal)
- **Sesgada a Derecha:** Media > Mediana (outliers altos)
- **Sesgada a Izquierda:** Media < Mediana (outliers bajos)

**Importancia:** Outliers legítimos vs errores de entrada → decisión sobre tratamiento.

---

### **Práctica 10: Segmentación de Clientes con Sentencias IF**

**Objetivo:** Aplicar reglas de negocio para clasificar clientes.

**Lógica Condicional:**

```python
# IF-ELSE simple
if cantidad_promedio > 22:
    print("Volumen Alto")
else:
    print("Volumen Bajo")

# IF-ELIF-ELSE (3 categorías)
if cantidad_mediana > 22:
    return "Volumen Alto"
elif cantidad_mediana >= 10:
    return "Volumen Medio"
else:
    return "Volumen Bajo"
```

**Métrica Clave:**
- Media de quantity: 32.36 → Clasificada como "Alto"
- Mediana de quantity: 14.00 → Clasificada como "Medio"
- **Conclusión:** La media está inflada por outliers

**Importancia:** Traduce métricas abstractas en categorías de negocio accionables.

---

### **Práctica 11: Segmentación Avanzada con np.where() y apply()**

**Objetivo:** Crear segmentaciones multidimensionales basadas en múltiples variables.

**Técnica 1: np.where() - Segmentación Binaria**

```python
df["volume_segment"] = np.where(
    df["quantity"] > 22, 
    "High Volume", 
    "Low Volume"
)
# Resultado: 1,261 (High) vs 3,739 (Low)
```

**Técnica 2: apply() con Función - Segmentación Multidimensional**

```python
def classify_volume(row):
    if pd.isna(row["customer_age"]) or pd.isna(row["quantity"]):
        return "Error en Datos"
    
    # Segmentación 2D: edad × volumen
    if row["quantity"] > 22:
        if row["customer_age"] > 55:
            return "Sr. High Volume"
        else:
            return "Jr. High Volume"
    else:
        if row["customer_age"] > 55:
            return "Sr. Low Volume"
        else:
            return "Jr. Low Volume"

df["volume_segment"] = df.apply(classify_volume, axis=1)
```

**Resultado: 4 Segmentos de Clientes**

| Segmento | Cantidad | % |
|----------|----------|-----|
| Jr. Low Volume | 3,004 | 60.1% |
| Sr. High Volume | 1,197 | 23.9% |
| Sr. Low Volume | 735 | 14.7% |
| Jr. High Volume | 64 | 1.3% |

**Interpretación de Negocio:**
- **Jr. Low Volume (Mayor):** Clientes jóvenes, compras pequeñas → Potencial para upsell
- **Sr. High Volume:** Clientes maduros, compras grandes → VIP, retención crítica
- **Jr. High Volume (Menor):** Clientes jóvenes, compras grandes → Segmento de alto valor emergente

**Caso Avanzado: Segmentación por Método de Pago**

```python
def classify_payment(row):
    card_payment = row["payment_method"] in ["credit_card", "debit_card"]
    high_volume = row["quantity"] > 22
    
    # Matriz 2×2
    if high_volume:
        return "card_high_volume" if card_payment else "no_card_high_volume"
    else:
        return "card_low_volume" if card_payment else "no_card_low_volume"
```

**Resultado: 4 Segmentos de Pago**

| Segmento | Cantidad |
|----------|----------|
| Card Low Volume | 2,704 |
| No-Card Low Volume | 1,035 |
| Card High Volume | 922 |
| No-Card High Volume | 339 |

**Insight:** 54.1% de transacciones usan tarjeta, pero solo el 35.1% son de alto volumen.

**Importancia:** Segmentación es la base de estrategias diferenciadas (marketing, pricing, servicio).

---

## 🛠️ Herramientas y Librerías Clave

| Librería | Funciones Principales |
|----------|----------------------|
| **pandas** | `read_csv()`, `.info()`, `.describe()`, `.groupby()`, `.apply()`, `.fillna()`, `.replace()` |
| **numpy** | `np.where()` — Segmentación condicional vectorizada |
| **Python nativo** | `if/elif/else` — Lógica condicional en funciones |

---

## 📈 Flujo Analítico Completo

```
┌─────────────────────────────────────────────────────────────┐
│                    DATASET CRUDO (5,008 × 11)               │
│                  everpeak_retail.csv                        │
└────────────┬────────────────────────────────────────────────┘
             │
      ┌──────▼──────────────────────────────────────────┐
      │  FASE 1: EXPLORACIÓN Y LIMPIEZA                 │
      │  ├─ Práctica 1: EDA Inicial (diagnóstico)       │
      │  ├─ Práctica 2: Análisis de Missingness         │
      │  ├─ Práctica 3: Funciones de Limpieza           │
      │  └─ Práctica 4: Pipeline Automatizado           │
      └────────┬─────────────────────────────────────────┘
               │
        ┌──────▼──────────────────┐
        │  DATASET LIMPIO         │
        │  everpeak_clean.csv     │
        └──────┬──────────────────┘
               │
      ┌────────▼──────────────────────────────────────┐
      │  FASE 2: ANÁLISIS ESTADÍSTICO                 │
      │  ├─ Prácticas 5-6: Estadísticas Descriptivas  │
      │  ├─ Prácticas 7-9: Distribuciones & Outliers  │
      │  └─ Prácticas 10-11: Segmentación             │
      └────────┬──────────────────────────────────────┘
               │
      ┌────────▼────────────────────────────────────┐
      │  INSIGHTS Y SEGMENTOS DE NEGOCIO             │
      │  ├─ 4 segmentos de edad×volumen              │
      │  ├─ 4 segmentos de pago×volumen              │
      │  └─ Métricas por categoría y geografía       │
      └─────────────────────────────────────────────┘
```

---

## 💡 Conceptos Clave Aprendidos

### 1. **Calidad de Datos**
- La limpieza consume 60-80% del tiempo analítico
- Documentar decisiones (sentinels, imputación) es crítico
- Crear flags preserva información sobre datos originales

### 2. **Estadística Descriptiva**
- Media es sensible a outliers → usar mediana para datos reales
- Desviación estándar mide variabilidad, no solo el promedio
- Frecuencias relativas (%) permiten comparaciones justas

### 3. **Automatización**
- Funciones + bucles → escalabilidad
- Pipelines → reproducibilidad
- apply() → procesamiento fila a fila flexible

### 4. **Segmentación**
- Reglas de negocio traducidas a código (if/elif/else)
- Multidimensional (2+ variables) → matrices de segmentos
- Cada segmento requiere estrategia diferenciada

---

## 📌 Dataset: Everpeak Retail

**Estructura:**
- **Período:** 2024
- **Registros:** 5,008 transacciones
- **Clientes únicos:** 1,829
- **Categorías de producto:** 8 (Fashion, Electronics, Beauty, Toys, Sports, Grocery, Home, unknown)
- **Métodos de pago:** 4 (Credit Card, PayPal, Debit Card, Cash)
- **Cobertura geográfica:** 10 ciudades, 9 estados (USA)

---

## 🎯 Resultados y Aplicaciones

### Segmentación Final por Volumen y Edad

| Segmento | Tamaño | Estrategia |
|----------|--------|-----------|
| **Jr. Low** | 3,004 | Educación, cross-sell, loyalty |
| **Sr. High** | 1,197 | VIP, servicio premium, retención |
| **Sr. Low** | 735 | Reactivación, ofertas especiales |
| **Jr. High** | 64 | Monitoreo como futuros VIP |

### Segmentación por Método de Pago

- **Card users** (80%): Mayor control, datos transaccionales ricos
- **No-card users** (20%): Efectivo/PayPal, menos datos pero importantes

---

## 📚 Competencias Desarrolladas

✅ Diagnóstico de calidad de datos (missingness, cardinalidad, tipos)  
✅ Automatización de pipelines de limpieza  
✅ Análisis estadístico descriptivo (univariado y bivariado)  
✅ Detección y manejo de outliers  
✅ Segmentación condicional (if/where/apply)  
✅ Interpretación de resultados para stakeholders comerciales  

---

## 🔗 Archivos del Sprint

```
📦 Sprint 7
├── 📄 Practica_1_Identificando_variables_relevantes_para_el_análisis.ipynb
├── 📄 practica_2_manejo_de_valores_ausentes_o_invalidos.ipynb
├── 📄 practica_3_creacion_de_funciones_y_bucles_para_la_limpiza.ipynb
├── 📄 practica_4_creacion_de_un_piperline_de_limpieza.ipynb
├── 📄 creacion_de_piperline_de_sesiones_de_estudio.ipynb
├── 📄 practica_5_media_estadistica_en_columnas_numericas.ipynb
├── 📄 practica_6_Medidas_Estadísticas_en_Columnas_Categóricas.ipynb
├── 📄 practica_7_Visualizando_distribuciones_con_Histogramas.ipynb
├── 📄 practica_8_Explorando_distribuciones_con_Boxplots_e_Histogramas.ipynb
├── 📄 PRACTICA_9_Identificando_valores_atípicos_con_reglas_estadísticas.ipynb
├── 📄 practica_10_Segmentación_de_clientes_con_sentencias_if.ipynb
└── 📄 practica_11_Segmentación_para_el_análisis_de_clientes.ipynb
```

---

## 🎓 Conclusión

Este sprint proporciona el **fundamento técnico** para análisis de datos profesional:

1. **De lo crudo a lo limpio:** Transformación metódica de datos
2. **De los números a los insights:** Interpretación estadística
3. **De los insights a la acción:** Segmentación operacional

La metodología aprendida es escalable a datasets de cualquier tamaño y dominio, estableciendo estándares de calidad y reproducibilidad esenciales en la industria.

---

**Autor:** Victor Manuel Lopez Prado  
**Institución:** TripleTen Data Analytics Sprint 7  
**Dataset:** Everpeak Retail (Simulado - Propósitos Educativos)  
**Última Actualización:** 2026
