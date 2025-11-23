# "Modelo Predictivo de Calidad del Aire PM2.5: Un Análisis de Series de Tiempo y Clasificación para la Predicción de Niveles de Contaminación"

---

## Introducción

La contaminación del aire por partículas finas (PM2.5) representa uno de los principales riesgos para la salud pública a nivel mundial. Las partículas PM2.5 son partículas en suspensión con un diámetro aerodinámico menor a 2.5 micrómetros, capaces de penetrar profundamente en el sistema respiratorio e incluso alcanzar el torrente sanguíneo, causando enfermedades cardiovasculares, respiratorias y cáncer de pulmón.

La Organización Mundial de la Salud (OMS) ha establecido estándares de calidad del aire que categorizan los niveles de PM2.5 según su impacto en la salud:

| Categoría | Rango PM2.5 (µg/m³) | Implicación para la salud |
|-----------|---------------------|---------------------------|
| Bueno | 0-35 | Calidad satisfactoria, riesgo mínimo |
| Moderado | 35-75 | Aceptable, posible riesgo para grupos sensibles |
| No Saludable | 75-150 | Efectos adversos para grupos sensibles |
| Peligroso | >150 | Efectos graves para toda la población |

En un contexto donde la industrialización, el tráfico vehicular y las condiciones meteorológicas interactúan de manera compleja, resulta fundamental desarrollar modelos predictivos que permitan anticipar episodios de contaminación y activar medidas preventivas.

Este análisis busca desarrollar un modelo que permita **predecir la categoría de calidad del aire para las próximas 24 horas**, utilizando técnicas de análisis de series de tiempo y clasificación supervisada.

---

## Los Datos

El dataset utilizado contiene **43,824 observaciones horarias** de calidad del aire y variables meteorológicas, abarcando un período de **5 años** (2010-2014).

Las variables registradas incluyen:

**Variable objetivo:**
- **pm2.5**: Concentración de partículas PM2.5 (µg/m³)

**Variables meteorológicas:**
- **DEWP**: Punto de rocío (°C) - Indicador de humedad atmosférica
- **TEMP**: Temperatura ambiente (°C)
- **PRES**: Presión atmosférica (hPa)
- **cbwd**: Dirección combinada del viento (NW, NE, SE, cv)
- **Iws**: Velocidad del viento acumulada (m/s)
- **Is**: Horas acumuladas de nieve
- **Ir**: Horas acumuladas de lluvia

**Variables temporales derivadas:**
- Año, mes, día, hora de medición

El dataset permite examinar la relación entre condiciones meteorológicas y niveles de contaminación, así como identificar patrones temporales (diarios, mensuales, anuales) en la concentración de PM2.5.

---

## 1. ¿Cuál es la distribución de la calidad del aire en el dataset?

### 1.1 Estadísticas descriptivas de PM2.5

El análisis de la concentración de PM2.5 revela una situación preocupante en términos de calidad del aire:

| Estadístico | Valor |
|-------------|-------|
| Media | 98.61 µg/m³ |
| Mediana | 72.00 µg/m³ |
| Desviación Estándar | 92.05 µg/m³ |
| Mínimo | 0.00 µg/m³ |
| Máximo | 994.00 µg/m³ |
| Percentil 25 | 29.00 µg/m³ |
| Percentil 75 | 137.00 µg/m³ |

La media (98.61 µg/m³) se encuentra **muy por encima** del límite recomendado por la OMS de 35 µg/m³ para exposición de 24 horas. La diferencia significativa entre media y mediana (98.61 vs 72.00) indica una distribución sesgada hacia la derecha, con episodios frecuentes de contaminación extrema que elevan el promedio.

El valor máximo de **994 µg/m³** representa un nivel catastrófico de contaminación, más de 28 veces el límite "bueno".

### 1.2 Distribución por categorías de calidad del aire

| Categoría | Registros | Porcentaje |
|-----------|-----------|------------|
| Bueno (≤35 µg/m³) | 12,120 | 29.0% |
| Moderado (35-75 µg/m³) | 9,391 | 22.5% |
| No Saludable (75-150 µg/m³) | 11,255 | 27.0% |
| Peligroso (>150 µg/m³) | 8,991 | 21.5% |

**HALLAZGO CRÍTICO:** Solo el **29%** del tiempo la calidad del aire es "Buena". Esto significa que durante el **71% del período estudiado**, la población estuvo expuesta a niveles de PM2.5 superiores a los recomendados por la OMS.

Particularmente alarmante es que el nivel "Peligroso" representa el **21.5%** de las observaciones - aproximadamente 1 de cada 5 horas presenta condiciones que pueden causar efectos graves para toda la población.

### 1.3 Episodios de contaminación

- **Total de días en el dataset:** 1,789 días
- **Días con al menos 1 hora "Peligroso":** 880 días (49.2%)
- **Número de episodios peligrosos:** 906 episodios
- **Duración promedio de episodios:** 9.9 horas
- **Episodio más largo:** 161 horas (6.7 días consecutivos)

Esto indica que prácticamente **la mitad de los días** experimentaron al menos una hora de contaminación peligrosa, y cuando ocurren estos episodios, tienden a persistir por casi 10 horas en promedio.

---

## 2. ¿Existen patrones temporales en la concentración de PM2.5?

### 2.1 Variación anual

| Año | PM2.5 Promedio (µg/m³) |
|-----|------------------------|
| 2010 | 104.05 |
| 2011 | 99.07 |
| 2012 | 90.55 |
| 2013 | 101.71 |
| 2014 | 97.73 |

No se observa una tendencia clara de mejora o empeoramiento a lo largo de los años. El año con mejor calidad del aire fue **2012** (90.55 µg/m³), mientras que **2010** presentó los niveles más altos (104.05 µg/m³). La variación interanual sugiere que factores meteorológicos anuales podrían tener mayor influencia que políticas de control de emisiones.

### 2.2 Variación mensual - PATRÓN ESTACIONAL IDENTIFICADO

| Mes | PM2.5 (µg/m³) | Clasificación |
|-----|---------------|---------------|
| Enero | 115.06 | Alto |
| **Febrero** | **125.74** | **Más alto** |
| Marzo | 97.76 | Medio-Alto |
| Abril | 83.71 | Medio |
| Mayo | 80.11 | Medio |
| Junio | 96.51 | Medio-Alto |
| Julio | 94.33 | Medio-Alto |
| **Agosto** | **80.00** | **Más bajo** |
| Septiembre | 85.21 | Medio |
| **Octubre** | **120.40** | **Alto** |
| Noviembre | 105.76 | Alto |
| Diciembre | 98.20 | Medio-Alto |

**PATRÓN ESTACIONAL CRÍTICO:**
- **Meses más contaminados:** Febrero (125.74), Octubre (120.40), Enero (115.06)
- **Meses más limpios:** Agosto (80.00), Mayo (80.11), Abril (83.71)

Este patrón es consistente con:
1. **Invierno más contaminado:** Mayor uso de calefacción + inversiones térmicas que atrapan contaminantes
2. **Otoño con pico secundario:** Quema de residuos agrícolas + condiciones de estabilidad atmosférica
3. **Verano más limpio:** Mayor ventilación atmosférica + ausencia de calefacción

### 2.3 Variación horaria - CICLO DIARIO

| Característica | Hora | PM2.5 (µg/m³) |
|----------------|------|---------------|
| **Hora más contaminada** | 1:00 AM | 113.70 |
| **Hora más limpia** | 3:00 PM | 85.53 |

El patrón diario muestra que la **madrugada** presenta los niveles más altos de PM2.5, mientras que la **tarde** tiene los niveles más bajos. Esto se explica por:

1. **Noches más contaminadas:**
   - Inversión térmica nocturna (aire frío cerca del suelo atrapa contaminantes)
   - Menor altura de la capa de mezcla atmosférica
   - Acumulación de emisiones del día

2. **Tardes más limpias:**
   - Mayor actividad convectiva (el sol calienta el suelo y genera mezcla vertical)
   - Mayor altura de la capa de mezcla
   - Mejor dispersión de contaminantes

---

## 3. ¿Cómo influyen las condiciones meteorológicas en la contaminación?

### 3.1 Correlaciones con variables meteorológicas

| Variable | Correlación con PM2.5 | Interpretación |
|----------|----------------------|----------------|
| **Iws (Viento)** | **-0.2478** | Mayor viento → Menor PM2.5 (dispersión) |
| DEWP (Punto rocío) | +0.1714 | Mayor humedad → Mayor PM2.5 |
| TEMP (Temperatura) | -0.0905 | Mayor temperatura → Menor PM2.5 |
| Ir (Lluvia) | -0.0514 | Lluvia → Menor PM2.5 (lavado atmosférico) |
| PRES (Presión) | -0.0473 | Alta presión → Menor PM2.5 |
| Is (Nieve) | +0.0193 | Correlación débil |

**HALLAZGO PRINCIPAL:** La **velocidad del viento** es el factor meteorológico más influyente (r = -0.25). Esto confirma que la dispersión mecánica de contaminantes por viento es el principal mecanismo de limpieza atmosférica.

### 3.2 PM2.5 según dirección del viento

| Dirección | PM2.5 Promedio | Interpretación |
|-----------|----------------|----------------|
| **cv (calma)** | **126.15 µg/m³** | Peor calidad - sin dispersión |
| SE (Sureste) | 110.82 µg/m³ | Alta - posible transporte de contaminantes |
| NE (Noreste) | 90.18 µg/m³ | Moderada |
| **NW (Noroeste)** | **70.13 µg/m³** | Mejor calidad - aire limpio |

**CONCLUSIÓN:** Los vientos del **Noroeste** traen aire más limpio, mientras que las condiciones de **calma** (cv) resultan en la peor calidad del aire. Los vientos del Sureste podrían transportar contaminantes desde zonas industriales o urbanas densas.

---

## 4. ¿La serie de PM2.5 es estacionaria?

### 4.1 Test de Dickey-Fuller Aumentado (ADF)

El test de estacionariedad es fundamental para determinar qué tipo de modelo de series de tiempo aplicar.

| Estadístico | Valor |
|-------------|-------|
| ADF | -21.3471 |
| P-valor | 0.000000 |
| Lags utilizados | 54 |

**Valores críticos:**
- 1%: -3.4305
- 5%: -2.8616
- 10%: -2.5668

**CONCLUSIÓN:** Con un estadístico ADF de -21.35 (mucho menor que todos los valores críticos) y un p-valor prácticamente cero, **rechazamos la hipótesis nula**. La serie de PM2.5 **ES ESTACIONARIA**.

Esto significa que:
- La media de la serie es constante en el tiempo
- La varianza es constante
- La covarianza entre períodos depende solo de la distancia temporal, no del momento

### 4.2 Análisis de Autocorrelación (Covarianza Temporal)

| Lag | Período | Autocorrelación |
|-----|---------|-----------------|
| 1 | 1 hora | 0.9661 |
| 6 | 6 horas | 0.7485 |
| 12 | 12 horas | 0.5683 |
| 24 | 1 día | 0.4024 |
| 48 | 2 días | 0.1561 |
| 168 | 1 semana | 0.0266 |

**INTERPRETACIÓN:**

1. **Alta autocorrelación a corto plazo (r=0.97 en 1 hora):** El valor de PM2.5 en un momento está fuertemente relacionado con el valor de la hora anterior. Esto indica "memoria" en la serie.

2. **Decaimiento gradual:** La correlación disminuye progresivamente con el tiempo, típico de un proceso autoregresivo (AR).

3. **Autocorrelación significativa hasta 24 horas:** La correlación de 0.40 a lag 24 confirma el ciclo diario identificado anteriormente.

4. **Pérdida de memoria a 1 semana (r=0.03):** Después de una semana, prácticamente no hay relación entre valores, lo que sugiere que las predicciones a largo plazo serán menos confiables.

**IMPLICACIÓN PARA MODELADO:** Un modelo **ARMA** (Autoregresivo de Media Móvil) es apropiado dado que:
- La serie es estacionaria (no necesita diferenciación, d=0)
- El componente AR es dominante (decaimiento lento de ACF)
- Existe estructura temporal aprovechable para predicción

---

## 5. ¿Qué tan bien podemos predecir la calidad del aire a 24 horas?

### 5.1 Configuración del modelo de clasificación

Para la predicción, se utilizó un **Pipeline** que integra:

1. **Preprocesamiento:**
   - Imputación de valores faltantes (mediana para numéricos, moda para categóricos)
   - Escalado estándar de variables numéricas
   - Codificación One-Hot de variables categóricas

2. **Variables predictoras (features):**
   - Meteorológicas: DEWP, TEMP, PRES, Iws, Is, Ir, cbwd
   - Temporales: hora, día de semana, mes
   - Lags de PM2.5: valores de 1, 6, 12 y 24 horas antes
   - Medias móviles: promedio de 6h y 24h anteriores

3. **División de datos (respetando orden temporal):**
   - Entrenamiento: 35,020 muestras (80%)
   - Prueba: 8,756 muestras (20%)

### 5.2 Comparación de modelos

| Modelo | Accuracy | Precision | Recall | F1-Score |
|--------|----------|-----------|--------|----------|
| **Gradient Boosting** | **0.4428** | **0.4405** | **0.4428** | **0.4327** |
| Random Forest | 0.4084 | 0.4016 | 0.4084 | 0.4024 |
| Logistic Regression | 0.3989 | 0.3952 | 0.3989 | 0.3731 |
| KNN | 0.3620 | 0.3654 | 0.3620 | 0.3633 |

**MEJOR MODELO: Gradient Boosting** con un Accuracy del 44.28%

### 5.3 Explicación de métricas

| Métrica | Definición | Valor Obtenido | Interpretación en este contexto |
|---------|------------|----------------|--------------------------------|
| **Accuracy** | Proporción de predicciones correctas | 44.28% | De 100 predicciones, ~44 son correctas |
| **Precision** | De las alertas emitidas, ¿cuántas son correctas? | 44.05% | De cada 100 alertas de una categoría, 44 son correctas |
| **Recall** | De los casos reales, ¿cuántos detectamos? | 44.28% | Detectamos 44 de cada 100 episodios reales |
| **F1-Score** | Balance entre Precision y Recall | 43.27% | Promedio armónico de ambas métricas |

### 5.4 Rendimiento por categoría

| Categoría | Precision | Recall | Interpretación |
|-----------|-----------|--------|----------------|
| Bueno | 50.62% | 44.76% | Detecta bien días buenos, pero con falsas alarmas |
| Moderado | 37.22% | 21.25% | **Peor rendimiento** - difícil de predecir |
| No Saludable | 40.52% | 54.01% | Buena detección de episodios no saludables |
| **Peligroso** | **47.05%** | **55.95%** | **Mejor detección de episodios críticos** |

**HALLAZGO IMPORTANTE PARA SALUD PÚBLICA:**
El modelo tiene el **mejor Recall (55.95%)** para la categoría "Peligroso", lo que significa que detecta correctamente más de la mitad de los episodios de contaminación severa. Esto es crucial porque en contextos de salud pública, **es preferible tener falsas alarmas que perder episodios peligrosos**.

### 5.5 Matriz de confusión

|  | Pred: Bueno | Pred: Moderado | Pred: No Salud. | Pred: Peligroso |
|--|-------------|----------------|-----------------|-----------------|
| **Real: Bueno** | **1,149** | 346 | 594 | 478 |
| **Real: Moderado** | 487 | **421** | 845 | 228 |
| **Real: No Salud.** | 418 | 299 | **1,319** | 406 |
| **Real: Peligroso** | 216 | 65 | 497 | **988** |

**Observaciones:**
1. La diagonal (en negrita) muestra las predicciones correctas
2. Los errores más comunes son entre categorías adyacentes (ej: "Moderado" confundido con "No Saludable")
3. Es raro confundir extremos ("Bueno" con "Peligroso": solo 478+216=694 casos de 8,756)

### 5.6 Variables más importantes para la predicción

| Variable | Importancia | Interpretación |
|----------|-------------|----------------|
| **pm2.5 (actual)** | **0.3385** | El valor actual es el mejor predictor del futuro |
| PRES (presión) | 0.1317 | Alta presión = estabilidad = más contaminación |
| DEWP (punto rocío) | 0.0881 | Humedad influye en formación de partículas |
| TEMP (temperatura) | 0.0731 | Afecta mezcla atmosférica |
| mes | 0.0626 | Captura estacionalidad |
| pm25_media_6h | 0.0517 | Tendencia reciente |
| Iws (viento) | 0.0440 | Factor de dispersión |
| pm25_media_24h | 0.0399 | Tendencia diaria |
| cbwd_NW | 0.0313 | Viento del noroeste = aire limpio |

---

## 6. Conclusiones

### 6.1 SITUACIÓN DE CALIDAD DEL AIRE - PREOCUPANTE

El análisis revela una situación crítica de contaminación:
- **71% del tiempo** la calidad del aire excede los límites recomendados
- La concentración promedio (98.61 µg/m³) es **casi 3 veces** el límite de 35 µg/m³
- **49.2% de los días** experimentaron al menos una hora con niveles "Peligrosos"
- Los episodios de contaminación severa duran en promedio **10 horas**

### 6.2 PATRONES TEMPORALES IDENTIFICADOS

| Patrón | Hallazgo | Implicación |
|--------|----------|-------------|
| **Estacional** | Febrero más contaminado, Agosto más limpio | Alertas reforzadas en invierno |
| **Diario** | 1 AM más contaminado, 3 PM más limpio | Actividades al aire libre preferibles en tarde |
| **Meteorológico** | Viento NW = aire limpio; Calma = peor calidad | Pronósticos de viento útiles para alertas |

### 6.3 ANÁLISIS DE SERIES DE TIEMPO

1. **La serie ES ESTACIONARIA** - confirmado por test ADF (p-valor < 0.001)
2. **Alta autocorrelación a corto plazo** - proceso AR dominante
3. **Memoria útil hasta 24 horas** - predicciones a corto plazo viables
4. **Ciclo diario evidente** - correlación significativa en lag 24

### 6.4 MODELO PREDICTIVO

El mejor modelo (**Gradient Boosting**) logra:
- **Accuracy: 44.28%** - mejor que azar (25% para 4 clases)
- **Recall para "Peligroso": 55.95%** - detecta más de la mitad de episodios críticos
- **Variable más importante: PM2.5 actual** (34% de importancia)

### 6.5 LIMITACIONES Y RECOMENDACIONES

**Limitaciones:**
- El accuracy del 44% indica que el problema es complejo
- La categoría "Moderado" es difícil de predecir (21% recall)
- Horizonte de 24 horas puede ser muy amplio

**Recomendaciones:**
1. **Reducir horizonte de predicción** a 6-12 horas para mejorar precisión
2. **Usar predicción probabilística** en lugar de categórica
3. **Incorporar más variables** (tráfico, actividad industrial, pronósticos meteorológicos)
4. **Para salud pública:** Priorizar alto Recall en categoría "Peligroso", aceptando más falsas alarmas

### 6.6 MÉTRICAS DE EVALUACIÓN - RESUMEN

| Métrica | Fórmula Conceptual | Uso |
|---------|-------------------|-----|
| **Accuracy** | Correctos / Total | Evaluación general |
| **Precision** | VP / (VP + FP) | Cuando el costo de falsas alarmas es alto |
| **Recall** | VP / (VP + FN) | Cuando perder casos reales es crítico |
| **F1-Score** | 2 × (P × R) / (P + R) | Balance entre ambos |

**En contexto de salud pública, RECALL es prioritario** - es preferible una falsa alarma de contaminación que no detectar un episodio peligroso real.

---

*Análisis realizado sobre datos de calidad del aire PM2.5 (2010-2014). Metodología: Series de tiempo (ARMA), Clasificación supervisada con Pipelines (Gradient Boosting). Métricas: Precision, Recall, Accuracy, F1-Score.*
