# 📊 Dashboard de Flujo de Caja (Cash Flow) 2024

## 📌 Descripción del Proyecto
Este proyecto consiste en un **Panel Financiero de Flujo de Caja (Cash Flow)** desarrollado en Power BI para el análisis de los movimientos de tesorería, cobros, pagos, saldos operativos e indicadores de liquidez correspondientes al año **2024**.

El objetivo principal es supervisar la posición líquida de la empresa, analizar los flujos por tipo (*Operaciones, Inversiones, Financiamiento*), controlar la evolución de cuentas por cobrar y por pagar, y evaluar ratios de endeudamiento neto y liquidez frente a metas establecidas.

---

## 📊 Vista Previa del Dashboard

![Dashboard Flujo de Caja](img/01.dashboard-flujo-caja.png)

---

## 🏗️ Modelo de Datos y Arquitectura
El proyecto implementa un modelo en estrella optimizado para análisis financiero y contable:

* **`Flujo_Caja`** (Tabla de Hechos): Contiene el detalle transaccional de ingresos, egresos, tipo de flujo, concepto, categoría contable e importe.
* **`Tabla_Calendario`** (Tabla de Dimensión): Dimensión de tiempo estandarizada para inteligencia temporal y agregaciones por mes.

![Modelo de Datos](img/02.modelado_datos.png)

---

## 🧮 Medidas DAX Implementadas

### 💵 Métricas Base y Saldos

```dax
Total Importe = SUM(Flujo_Caja[Importe])

Total Ingresos = CALCULATE([Total Importe], Flujo_Caja[Categoría] = "Ingreso")

Total Egresos = CALCULATE([Total Importe], Flujo_Caja[Categoría] = "Egreso")

Saldo General = [Total Ingresos] - [Total Egresos]

Efectivo al Final del Periodo = [Total Ingresos] - [Total Egresos]

Saldo Operativo = 
VAR Ingresos_Operacionales = CALCULATE([Total Importe], Flujo_Caja[Categoría] = "Ingreso", Flujo_Caja[Tipo] = "Operaciones")
VAR Egresos_Operacionales = CALCULATE([Total Importe], Flujo_Caja[Categoría] = "Egreso", Flujo_Caja[Tipo] = "Operaciones")
RETURN
    Ingresos_Operacionales - Egresos_Operacionales

% Total (Importe) = 
DIVIDE(
    [Total Importe],
    CALCULATE([Total Importe], ALLSELECTED())
)

% Total de los Tipos de Flujo = 
DIVIDE(
    [Efectivo al Final del Periodo],
    CALCULATE([Efectivo al Final del Periodo], ALLSELECTED())
)
```

### 📉 Control de Cuentas, Endeudamiento y Ratios

```dax
Saldo Cuentas por Cobrar = 
VAR Cuenta_Cobrada = CALCULATE([Total Importe], Flujo_Caja[Concepto] = "Cuenta Cobrada")
VAR Cuentas_por_Cobrar = CALCULATE([Total Importe], Flujo_Caja[Concepto] = "Cuentas por Cobrar")
RETURN
    Cuentas_por_Cobrar - Cuenta_Cobrada

Saldo Cuentas por Pagar = 
VAR Cuentas_Pagar_Proveedores = CALCULATE([Total Importe], Flujo_Caja[Concepto] = "Cuentas por Pagar(Proveedores)")
VAR Pago_Proveedores = CALCULATE([Total Importe], Flujo_Caja[Concepto] = "Pago a Proveedores")
RETURN
    Cuentas_Pagar_Proveedores - Pago_Proveedores

Endeudamiento Neto (Banco) = 
VAR Amortizacion_Prestamo = CALCULATE([Total Importe], Flujo_Caja[Concepto] = "Amortización de Préstamos")
VAR Deuda_Largo_Plazo = CALCULATE([Total Importe], Flujo_Caja[Concepto] = "Deuda a largo Plazo")
RETURN
    Deuda_Largo_Plazo - Amortizacion_Prestamo

Ratio de Efectivo = 
VAR Efectivo_Final_Periodo = CALCULATE([Saldo General], Flujo_Caja[Tipo] IN {"Financiamiento", "Inversiones", "Operaciones"})
VAR Cuentas_por_Pagar = CALCULATE([Total Importe], Flujo_Caja[Concepto] = "Cuentas por Pagar(Proveedores)")
VAR Pago_Proveedores = CALCULATE([Total Importe], Flujo_Caja[Concepto] = "Pago a Proveedores")
VAR Pasivo_Corriente = Cuentas_por_Pagar - Pago_Proveedores
RETURN
    DIVIDE(Efectivo_Final_Periodo, Pasivo_Corriente, 0)

Proporcion Egresos/Ingresos = DIVIDE([Total Egresos], [Total Ingresos], 0)

Meta Ingresos = 100000

Meta Egresos = 50000
```

### 📌 Mínimos y Máximos Dinámicos

```dax
Valor Maximo = 
VAR TablaMax = 
    CALCULATETABLE(
        ADDCOLUMNS(
            SUMMARIZE(Flujo_Caja, Tabla_Calendario[Mes]), 
            "SaldoMax", [Saldo General]
        ),
        ALLSELECTED()
    )
VAR Max_Saldo = MAXX(TablaMax, [SaldoMax])
RETURN
    IF(Max_Saldo = [Saldo General], Max_Saldo, BLANK())

Valor Minimo = 
VAR TablaMin = 
    CALCULATETABLE(
        ADDCOLUMNS(
            SUMMARIZE(Flujo_Caja, Tabla_Calendario[Mes]), 
            "SaldoMin", [Saldo General]
        ),
        ALLSELECTED()
    )
VAR Min_Saldo = MINX(TablaMin, [SaldoMin])
RETURN
    IF(Min_Saldo = [Saldo General], Min_Saldo, BLANK())
```

---

## 🛠️ Herramientas y Tecnologías

* **Power BI Desktop:** Transformación de datos, diseño de interfaz con lienzo personalizado SVG (fondo-dashboard.svg) y tarjetas dinámicas.

* **DAX (Data Analysis Expressions):** Indicadores de liquidez, métricas relativas (ALLSELECTED), análisis operacionales y resaltado de picos (máx/mín).

* **Excel / Power Query:** Modelado de origen de datos estandarizado (Datos-Flujo de Caja(Cash Flow).xlsx).