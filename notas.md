#Kusto
## Indice
* [Operadores más usados](#Operadores-más-usados)
    * [Search](#Search)
    * [Where](#Where)
    * [Take o Limit](#Take-o-Limit)
## Operadores más usados
### Search
El operador search realiza una búsqueda textual sobre todas los elementos que marquemos, si no lo marcamos, lo hará sobre toda la base de datos.
Para buscar en todos los campos de una tabla, se puede colocar delante de una tuberia.

```
Perf
| search "memory"
```

También se puede usar el operador "in", son dos consultas equivalentes.
```
search in (Perf, Event, Alert) "memory"
```
El operador por defecto no es sensitivo. Para hacerlo:
```
Perf
| search kind=case_sensitive "memory"
```
Se puede usar el operador search especificando la columna sobre la que se quiere buscar y "==" para coincidencia exacta.
```
Perf
| search CounterName=="Available MBytes"
```
Para coincidencia parcial se deberá usar ":"
```
Perf
| search CounterName:"MBytes"
```
Se pueden usar wildcards.
```
Perf
| search "*Bytes*"

Perf
| search * startswith "Bytes"

Perf
| search * endswith "Bytes"

Perf
| search "Free*Bytes"
```
Cuando se utilizan los operadores lógicos, estos se aplican al ambito de la consulta, no a un único campo.  
En el ejemplo, no tienen que coincidir ambos operadores en el mismo campo si no en la misma fila. 
```
Perf
| search "Free*Bytes" and ("C:" or "D:")
```
Expresiones regulares, poco eficientes.
```
Perf
| search InstanceName matches regex "[A-Z]:"
```
### Where
Al igual que search sirve para filtrar resultados, pero se utiliza para filtrar por límites basados en condiciones.  
Cuando hacemos búsquedas basadas en tiempo, cambia el filtro de tiempo de la consola.
Los valores para las unidades de tiempo son:
* d - days
* h - hours
* m - minutes
* s - seconds
* ms - milliseconds
* microsecond - microseconds
```
Perf
| where TimeGenerated >= ago(1h)
```
Uso de operadores lógicos en where
```
Perf
| where TimeGenerated >= ago(1h)
    and CounterName == "Bytes Received/sec"

Perf 
| where TimeGenerated  >= ago(1h)
    and (CounterName == "Bytes Received/sec"
         or
         CounterName  == "% Processor Time"
        )
    and CounterValue > 0
```
Uso de varios operadores where consecutivos
```
Perf 
| where TimeGenerated  >= ago(1h)
| where (CounterName == "Bytes Received/sec"
         or
         CounterName  == "% Processor Time"
        )
| where CounterValue > 0
```
Se puede utilizar el where para consultas similares a search, tarda lo mismo.
```
Perf
| where * has "Bytes"
// Iguales
Perf
| where * contains "Bytes"

Perf
| where * hasprefix "Bytes" 

Perf
| where * hassuffix "Bytes"

Perf
| where * contains "Bytes"

Perf
| where InstanceName matches regex "[A-Z]:"
```
## Take o Limit
Se usa para que nos devuelva un número concreto de resultados aleatorios.  
Cada vez que se ejecuta la consulta los resultados pueden ser diferentes.  
Muy práctico para saber si una consulta devolvera resultados correctos sin tener que esperar por el resultado completo.  
```
Perf
| take 10

Perf
| limit 10
```
## Count
Este operador devulve el número de filas que coinciden con los criterios marcados.
```
Perf
| count

Perf 
| where TimeGenerated  >= ago(1h)
    and CounterName == "Bytes Received/sec"
    and CounterValue > 0
| count
```
## Summarize
Sirve para agrupar resultados. Se pueden marcar varios para conjuntos.  
```
Perf 
| summarize count() by ObjectName, CounterName

Perf 
| where CounterName == "% Free Space"
| summarize NumberOfEntries=count() //asignación de alias
          , AverageFreeSpace=avg(CounterValue) 
         by CounterName
```
El la función bin() permite generar intervalos marcando una unidad del intervalo.  
```
Perf 
| summarize NumberOfEntries=count() 
         by CounterName // Primera agrupación
          , bin(TimeGenerated, 1d) // Segunda agrupación por intervalos

Perf
| where CounterName == "% Free Space"
| summarize NumberOfRowsAtThisPercentLevel=count() 
         by bin(CounterValue,10) // Intervalos númericos en grupos de 10
```
## Extend
Permite crear columnas en el resultado. Sirve para crear columnas con campos calculados por ejemplo.
```
Perf 
| where CounterName == "Free Megabytes"
| extend FreeGB = CounterValue / 1000
       , FreeMB = CounterValue 
       , FreeKB = CounterValue * 1000
```
La función strcat() permite concatenar strings.
```
Perf
| extend ObjectCounter = strcat(ObjectName, " - ", CounterName) 
```