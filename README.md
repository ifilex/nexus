# Nexus Crypto

### Señales, acumulación y coberturas configurables

Nexus Crypto es un Expert Advisor desarrollado en **MQL4 para MetaTrader 4**. Este repositorio comparte el código fuente de la **versión 1.01** para estudio, revisión y experimentación.

Permite combinar distintos modos de entrada, entradas adicionales y coberturas dentro de una estructura multiactivo.

## Contexto del proyecto

Como neuropsicólogo y desarrollador, me interesa explorar cómo se transforman observaciones en reglas de decisión y cómo esas reglas modifican la exposición ante resultados adversos.

Nexus permite examinar estas decisiones por separado: cuándo entrar, cuándo agregar posiciones, cuándo intentar cubrirlas y cómo gestionar las salidas.

Es un sistema basado en reglas. No incorpora aprendizaje automático ni un modelo neuropsicológico.

## Modos de entrada

| Modo | Implementación |
|---|---|
| `ENTRADA_RSI_EMA` | RSI en sobreventa o sobrecompra, posición del precio respecto de la EMA de tendencia y relación entre EMA rápida y lenta |
| `ENTRADA_BOLLINGER` | Contacto o superación de las bandas con condiciones de RSI |
| `ENTRADA_TENDENCIA` | Posición respecto de la EMA de tendencia, relación entre medias y límites de RSI |
| `ENTRADA_AGRESIVA` | Condiciones alternativas de RSI, tendencia y distancia respecto de la EMA rápida |

Los indicadores utilizan el período del gráfico y la vela en formación.

Aunque la interfaz menciona “cruce EMA”, el código evalúa la posición relativa actual de las medias; no comprueba un cruce entre dos velas.

El modo denominado “Solo Tendencia EMA200” también utiliza las medias rápida y lenta y el RSI.

## Entradas adicionales

Los nombres “recuperación” y “DCA” describen los modos del programa. No garantizan recuperar pérdidas.

### Martingala

Agrega posiciones cuando el precio se desplaza en contra una distancia configurada.

El lote adicional se calcula a partir del lote anterior multiplicado por `MultiplicadorLote`, cuyo valor predeterminado es **1.3**.

### DCA suave

Utiliza `LoteFijo` para las entradas adicionales, sujeto al mínimo del instrumento.

El lote de la entrada inicial puede calcularse por riesgo, por lo que no necesariamente coincide con el de los añadidos.

### DCA con indicador

Utiliza el mismo criterio de distancia y lote fijo, agregando una condición de RSI:

- Compras adicionales: RSI inferior a 40.
- Ventas adicionales: RSI superior a 60.

En los tres modos, las entradas adicionales heredan el stop registrado en la operación de referencia y se envían inicialmente sin take profit.

## Coberturas

### Automática por pérdida

Puede abrir una posición contraria sobre el mismo símbolo cuando:

- Hay al menos tres operaciones principales del lado afectado.
- Su pérdida flotante supera el porcentaje configurado del balance.
- No existe una cobertura identificada sobre ese símbolo.

El ratio predeterminado es **0.6** del lotaje acumulado de ese lado.

Las coberturas se abren sin stop loss ni take profit. En este modo puede activarse su cierre con beneficio mediante una rutina específica.

### Por pares asociados

El modo `HEDGE_CORRELACION` utiliza una tabla fija:

| Par base | Par asociado |
|---|---|
| BTCUSD | ETHUSD |
| ETHUSD | SOLUSD |
| SOLUSD | AVAXUSD |
| XRPUSD | ADAUSD |
| DOGEUSD | XRPUSD |
| LINKUSD | DOTUSD |

La búsqueda admite ambos sentidos y devuelve la primera coincidencia disponible.

**No calcula correlación histórica, estabilidad de la relación ni equivalencia monetaria entre contratos.**

La implementación abre ventas sobre el instrumento asociado cuando existen al menos tres compras principales en el símbolo de origen. No incluye una ruta equivalente para cubrir grupos de ventas.

### Sin cobertura y manual

`HEDGE_NINGUNO` y `HEDGE_MANUAL` no ejecutan las rutinas automáticas de cobertura.

El modo manual no incorpora botones ni un flujo específico para crear coberturas desde el EA.

## Gestión de salida

El programa ofrece tres opciones:

- `SALIDA_TP_SL`: conserva la lógica individual.
- `SALIDA_GRUPAL`: habilita la rutina de modificación del TP conjunto.
- `SALIDA_MIXTO`: también habilita esa rutina.

En esta versión, los dos últimos modos comparten el mismo bloque de ejecución.

Las entradas iniciales reciben los TP y SL calculados independientemente del modo de salida. La rutina grupal modifica take profits; no implementa un cierre directo por beneficio monetario agregado.

El trailing de las posiciones principales se ejecuta en todos los modos, con una distancia de **12 unidades de pip según la conversión interna**. No se aplica a las coberturas.

## Filtros

Las entradas iniciales pueden condicionarse por:

- Horario del servidor.
- Exclusión de fines de semana.
- Volumen de ticks.
- Spread.
- Rutina de detección de barridos.

Estos filtros no se vuelven a exigir para las entradas adicionales ni para las coberturas.

La rutina de barridos requiere corrección, como se detalla en las limitaciones.

## Bloqueo por drawdown

Compara el equity de toda la cuenta con el balance registrado al inicializar el EA.

Al alcanzar el umbral:

- Bloquea nuevas entradas iniciales.
- Bloquea nuevas entradas de acumulación.
- Suspende las rutinas de cobertura.

**No cierra las posiciones existentes.**

La gestión del TP grupal y el trailing de posiciones principales continúan, si el procesamiento del símbolo llega a esas rutinas.

El cierre con beneficio de coberturas automáticas también queda suspendido, porque está dentro de la rutina de cobertura bloqueada.

El estado se restablece al reinicializar el EA.

## Configuración predeterminada

| Parámetro | Valor |
|---|---|
| Versión | `1.01` |
| `NumeroMagico` | `5500` |
| `ModoEntrada` | `ENTRADA_RSI_EMA` |
| `ModoHedge` | `HEDGE_AUTOMATICO` |
| `ModoRecuperacion` | `RECUP_DCA_SUAVE` |
| `ModoSalida` | `SALIDA_MIXTO` |
| Activos activados | BTCUSD, ETHUSD y SOLUSD |
| `MaxParesSimultaneos` | `3` |
| `RSIPeriodo` | `14` |
| RSI sobreventa / sobrecompra | `35 / 65` |
| EMA rápida / lenta / tendencia | `20 / 50 / 200` |
| Bollinger | `20 períodos, 2 desviaciones` |
| `UsarLotePorRiesgo` | `true` |
| `RiesgoPorcentaje` | `1.5` |
| `LoteFijo` | `0.01` |
| `MultiplicadorLote` | `1.3` |
| `DistanciaGridPips` | `20.0` |
| `MaxOrdenesPorLado` | `20` |
| `HedgeRatio` | `0.6` |
| `HedgeActivacionPct` | `5.0` |
| `HedgeCierrePips` | `10.0` |
| ATR: período / SL / TP | `14 / 2 / 4` |
| `MaxDrawdownPct` | `25.0` |
| `MaxRiesgoTotalPct` | `8.0`, sin aplicación efectiva |
| `SpreadMaximoPips` | `60.0` |
| Horario inicial | `08:00–22:00` |
| Exclusión de fines de semana | Activada |

Los valores describen el archivo publicado. No son recomendaciones.

## Instrumentos disponibles

La selección contempla:

BTCUSD, ETHUSD, SOLUSD, XRPUSD, ADAUSD, DOGEUSD, LTCUSD, BNBUSD, DOTUSD, LINKUSD, AVAXUSD, MATICUSD, BCHUSD y XLMUSD.

La disponibilidad depende del bróker. El programa no incluye conexión directa a exchanges ni gestión de billeteras.

## Instalación para pruebas

1. Abrir MetaTrader 4 en una cuenta demo.
2. Seleccionar **Archivo → Abrir carpeta de datos**.
3. Copiar `NEXUS_CRYPTO.mq4` dentro de `MQL4/Experts`.
4. Abrir el archivo en MetaEditor y compilar.
5. Revisar errores y advertencias.
6. Actualizar la lista de Expert Advisors.
7. Cargar el EA en un gráfico.
8. Comprobar símbolos, históricos, parámetros y permisos.

El procesamiento depende de los ticks del gráfico donde está instalado.

La cobertura sobre el mismo símbolo requiere que la cuenta permita posiciones opuestas.

Utiliza tres números mágicos consecutivos: `NumeroMagico`, `NumeroMagico + 1` y `NumeroMagico + 2`. Evitar colisiones con otras instancias.

## Estado y limitaciones conocidas

Puntos identificados mediante lectura del código:

- **Riesgo total:** `MaxRiesgoTotalPct` no se aplica como condición de apertura. La función de exposición existente tampoco se utiliza para imponer ese límite.
- **Filtro de barridos:** incluye en el rango la misma vela que compara. Con datos coherentes, las condiciones estrictas de ruptura no pueden cumplirse.
- **Cobertura cruzada:** comprueba si existe cobertura en el símbolo de origen, pero abre en otro símbolo. Puede repetir aperturas en ticks posteriores.
- **Activos de cobertura:** el par asociado puede no estar habilitado en la selección principal.
- **Lotaje cruzado:** utiliza lotes y precisión del instrumento de origen sin equiparar el valor económico de ambos contratos.
- **Cierre de coberturas:** la rutina de cierre con beneficio se llama desde el modo automático, no desde el modo por pares asociados.
- **Objetivo grupal:** las fórmulas convierten beneficio monetario a precio sin una conversión completa mediante el valor del tick. La fórmula compradora tampoco incorpora un precio de referencia.
- **Beneficio de cobertura en pips:** la conversión requiere revisión y protección frente a valores de tick nulos.
- **Límite de símbolos:** el conteo no se actualiza tras cada apertura dentro del recorrido.
- **Última operación:** se toma según el orden de recorrido, sin selección explícita por fecha.
- **Lotes DCA:** falta ajuste integral al máximo y paso permitido por el bróker.
- **Stops iniciales:** si no se supera la comprobación de distancia mínima, la orden puede enviarse con SL o TP en cero.
- **Datos y ejecución:** faltan validaciones integrales de históricos, parámetros y resultados de varias solicitudes.
- **Opciones visuales:** `MostrarNiveles` y `MostrarSenales` están declaradas, pero no tienen una implementación asociada en esta versión.
- **Panel:** el P/L mostrado como total corresponde a las posiciones principales del símbolo del gráfico; excluye coberturas y otros símbolos.

## Riesgo

Agregar posiciones ante movimientos adversos aumenta la exposición, incluso cuando los lotes adicionales son fijos.

Las coberturas pueden agregar costes, margen requerido y nuevas pérdidas. Los instrumentos asociados pueden cambiar su relación de precios.

El porcentaje usado para dimensionar una entrada no representa un límite de riesgo para toda la cartera.

## Resultados y validación

Este repositorio no incluye un historial auditado ni una validación independiente de rendimiento.

La documentación se basa en la lectura del código. No acredita compilación, ejecución ni resultados de backtesting.

## Autor

Desarrollado por **Emanuel**, neuropsicólogo y desarrollador del proyecto **WineBOX**.

- https://winebox.cloud
- https://wineboxtool.cloud

## Contribuciones

Se reciben reportes de errores y propuestas mediante Issues y Pull Requests.

Incluir instrumento, período del gráfico, parámetros, versión de MetaTrader 4 y mensajes relevantes del registro, sin credenciales ni datos personales.

## Licencia

Este proyecto se distribuye bajo la licencia **Apache License 2.0**.

Consultar el archivo [LICENSE](LICENSE) para conocer los términos completos.

## Aviso

Material técnico y educativo. No constituye asesoramiento financiero ni una promesa de resultados.
