#Kusto
## Indice
* [Operadores más usados](#Operadores-más-usados)
    * [Search](#Search)
    * [Where](#Where)
    * [Take o Limit](#Take-o-Limit)
    * [Count](#Count)
    * [Summarize](#Summarize)
    * [Extend](#Extend)
    * [Project](#Project)
    * [Disctinct](#Distinct)
    * [Top](#Top)
    * [Ejemplo](#Ejemplo)
* [Operadores escalares](#Operadores-escalares)
    * [Print](#Print)
    * [Ago](#Ago)
    * [Sort by](#Sort-by)
    * [Extract](#Extract)
    * [Parse](#Parse)
    * [Datetime / timespan](#Datetime-/-timespan)
    * [Format date](#Format-date)
    * [Start of time](#startofday-y-endofday-//week,-month,-year)
    * [Between](#Between)
    * [Datetime part](#Datetime-part)
    * [Todynamic](#Todynamic)
    * [Iif](#Iif)
    * [Case](#Case)
    * [Isempty e Isnull](#Isempty-e-Isnull)
    * [Split](#Split)
    * [String Operators](#String-Operators)
* [Agregaciones](#Agregaciones)
    * [Maseket() y makelist()](#Maseket()-y-makelist())
    * [Mxexpand()](#Mxexpand())
    * [Dcount()](#Dcount())
    * [Pivot()](#Pivot())
* [Datasets()](#Datasets)
    * [Let](#Let)
    * [Join](#Join)
    * [Union](#Union)
## Operadores más usados
### Search
El operador search realiza una búsqueda textual sobre todos los elementos que marquemos, si no lo marcamos, lo hará sobre toda la base de datos.
Para buscar en todos los campos de una tabla, se puede colocar delante de una tubería.

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
Cuando se utilizan los operadores lógicos, estos se aplican al ámbito de la consulta, no a un único campo.  
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
### Take o Limit
Se usa para que nos devuelva un número concreto de resultados aleatorios.  
Cada vez que se ejecuta la consulta los resultados pueden ser diferentes.  
Muy práctico para saber si una consulta devolverá resultados correctos sin tener que esperar por el resultado completo.  
```
Perf
| take 10

Perf
| limit 10
```
### Count
Este operador devuelve el número de filas que coinciden con los criterios marcados.
```
Perf
| count

Perf 
| where TimeGenerated  >= ago(1h)
    and CounterName == "Bytes Received/sec"
    and CounterValue > 0
| count
```
Está la opción `countif()` en al que únicamente contará si cumplimos con la condición.

### Summarize
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
         by bin(CounterValue,10) // Intervalos numéricos en grupos de 10
```
### Extend
Permite crear columnas en el resultado. Sirve para crear columnas con campos calculados, por ejemplo.
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
### Project
Este operador nos permite seleccionar las columnas deseadas de una tabla o resultad.  
```
Perf
| project ObjectName
        , CounterName 
        , InstanceName 
        , CounterValue
```
Project puede simular el operador Extend.
```
Perf
| where CounterName == "Free Megabytes"
| project ObjectName 
        , CounterName 
        , InstanceName 
        , TimeGenerated 
        , FreeGB = CounterValue / 1000
        , FreeMB = CounterValue 
        , FreeKB = CounterValue * 1000
```
Hay una variación del operador project llamado "project-away", que nos permite excluir columnas del resultado.  
Otra variación es "project-rename" que nos permite renombrar una columna en el resultado.
```
Perf
| project-away ObjectName

Perf
| project-rename ObjectName = Object
```
### Distinct
Este operador se usa para devolver valores únicos utilizando las columnas que se le pasen.  
```
Perf
| distinct ObjectName, CounterName

Event
| where EventLevelName == "Error"
| distinct Source
```
### Top
Nos devuelve los primeros N resultados de una lista ordenada.  
````
Perf
| top 20 by TimeGenereated desc
````
### Ejemplo
En este ejemplo se listan los equipos con poco espacio libre en disco.  
```
Perf
| where CounterName == "Free Megabytes"  // Get the Free Megabytes
    and TimeGenerated >= ago(1h)         // ...within the last hour
| project Computer                       // For each return the Computer Name
        , TimeGenerated                  // ...and when the counter was generated
        , CounterName                    // ...and the Counter Name
        , FreeMegabytes=CounterValue     // ...and rename the counter value
| distinct Computer                      // Now weed out duplicate rows as
         , TimeGenerated
         , CounterName                   // ...the perf table will have
         , FreeMegabytes                 // ...multiple entries during the day
| top 25 by FreeMegabytes asc            // Filter to most critical ones
```
## Operadores escalares
Estos operadores permiten modificar o trabajar con los datos. También incluyen operadores lógicos.
### Print
Sirve para imprimir un dato.
```
print Result = 21 * 2

print now()
```
### Ago
Operador para expresar un momento en el tiempo utilizando de referencia el momento actual.
```
print ago(1h)
```
Para expresar momentos futuros, únicamente hay que poner la cantidad de tiempo en negativo.
```
print ago(-365d)
```
### Sort by
Permite ordenar los resultados por una o varias columnas indicadas. Por defecto el sentido es descendente.
```
Perf
| where TimeGenerated > ago(15m)
| where CounterName == "Avg. Disk sec/Read"
    and InstanceName == "C:"
| project Computer
        , TimeGenerated 
        , ObjectName
        , CounterName 
        , InstanceName 
        , CounterValue 
| sort by Computer asc, TimeGenerated // Primero ordena por Computer
```
### Extract
Permite extraer una sub-cadena de una celda utilizando un expresión regular.
```
Perf
| where ObjectName == "LogicalDisk"
    and InstanceName matches regex "[A-Z]:"
| project Computer 
        , CounterName 
        , extract("[A-Z]:", 0, InstanceName) // El segundo parámetro es el grupo en caso de existir, 0 es para devolver todos.
```
### Parse
Sirve para dividir el contenido de un campo en otros utilizando los identificadores oportunos.
 ```
//Event code: 3005  Event message: An unhandled exception has occurred.  Event time: 4/1/2018 11:44:43 PM  Event time (UTC): 4/1/2018 11:44:43 PM  Event ID: b8eea7d21d044fa8a0a774392f1f5b24  Event sequence: 5892  Event occurrence: 217  Event detail code: 0    Application information:      Application domain: /LM/W3SVC/2/ROOT-2-131670348328019759      Trust level: Full      Application Virtual Path: /      Application Path: C:\inetpub\ContosoRetail\      Machine name: CONTOSOWEB1    Process information:      Process ID: 13200      Process name: w3wp.exe      Account name: IIS APPPOOL\ContosoRetail    Exception information:      Exception type: FormatException        
// ... more data followed
Event
| where RenderedDescription startswith "Event code:" 
| parse RenderedDescription with "Event code: " myEventCode 
                                 " Event message: " myEventMessage 
                                 " Event time: " myEventTime 
                                 " Event time (UTC): " myEventTimeUTC 
                                 " Event ID: " myEventID 
                                 " Event sequence: " myEventSequence
                                 " Event occurrence: " *
| project myEventCode 
        , myEventMessage 
        , myEventTime 
        , myEventTimeUTC 
        , myEventID 
        , myEventSequence 
```
### Datetime / timespan
Se puede utiliar la función datetime para expresar fechas o momentos.
```
Perf
| where CounterName == "Avg. Disk sec/Read"
| where CounterValue > 0
| take 100                       // done just to give us a small dataset to demo
| extend HowLongAgo=( now() - TimeGenerated )
       , TimeSinceStartOfYear=(TimeGenerated - datetime(2018-01-01)) // Tiempos en dias.horas:minutos... desde una fecha
| project Computer 
        , CounterName
        , CounterValue  
        , TimeGenerated 
        , HowLongAgo 
        , TimeSinceStartOfYear 
```
Para tener un resultado en una unidad de tiempo concreta, hay que dividir el resultado entre la unidad deseada.
```
TimeSinceStartOfYear=( TimeGenerated - datetime(2018-01-01) ) / 1d  // Expresado en dias
```
### Format date
Esta función sirve para formatear una fecha en el formato deseado.  
Se puede formatear tanto un datetime como un timespan.
```
Perf
| take 100                       // done just to give us a small dataset to demo
| project CounterName 
        , CounterValue 
        , TimeGenerated 
        , format_datetime(TimeGenerated, "y-M-d") // 20-5-30 // format_timespan
        , format_datetime(TimeGenerated, "yyyy-MM-dd")
        , format_datetime(TimeGenerated, "MM/dd/yyyy")
        , format_datetime(TimeGenerated, "MM/dd/yyyy hh:mm:ss tt") // 30/05/2020 03:49:58 AM
        , format_datetime(TimeGenerated, "MM/dd/yyyy HH:mm:ss")
        , format_datetime(TimeGenerated, "MM/dd/yyyy HH:mm:ss.ffff") // Con milisegundos
        
// Supported syntax - one letter is single number, two letters two numbers
//    d - Day, 1 to 31
//   dd - Day, 01 to 31
//    M - Month, 1 to 12
//   MM - Month, 01 to 12
//    y - Year, 0 to 9999
//   yy - Year, 00 to 9999
// yyyy - Year, 0000 to 9999
```
### startofday y endofday //week, month, year
Esta función sirve para calcular el inicio o el fin del día/semana... de una fecha dada.  
```
Event
| where TimeGenerated >= ago(7d)
| extend DayGenerated = startofday(TimeGenerated)
| project Source
        , DayGenerated 
| summarize EventCount=count() // Eventos generados por dia/fuente
         by DayGenerated
         , Source
```
### Between
Esta función se utiliza como filtro de un valor si este se encuentra entre dos dados.
```
Perf
| where CounterName == "% Free Space" 
| where CounterValue between ( 70.0 .. 100.0 )

Perf
| where CounterName == "% Free Space" 
| where TimeGenerated between ( startofday(datetime(2018-04-01)) .. endofday(datetime(2018-04-03)) )
```
Es posible negar esta función con "!"
```
Perf
| where CounterName == "% Free Space" 
| where CounterValue !between ( 70.0 .. 100.0 )
```
### Datetime part
Esta función permite extraer una parte de un objeto datetime.
```
Perf
| take 100
| project CounterName 
        , CounterValue 
        , TimeGenerated 
        , year = datetime_part("year", TimeGenerated)
        , quarter = datetime_part("quarter", TimeGenerated)
        , month = datetime_part("month", TimeGenerated)
        , weekOfYear = datetime_part("weekOfYear", TimeGenerated)
        , day = datetime_part("day", TimeGenerated)
        , dayOfYear = datetime_part("dayOfYear", TimeGenerated)
        , hour = datetime_part("hour", TimeGenerated)
        , minute = datetime_part("minute", TimeGenerated)
        , second = datetime_part("second", TimeGenerated)
        , millisecond = datetime_part("millisecond", TimeGenerated)
        , microsecond = datetime_part("microsecond", TimeGenerated)
        , nanosecond = datetime_part("nanosecond", TimeGenerated)

Event // Eventos agrupados por horas del día
| where TimeGenerated >= ago(7d)
| extend HourOfDay = datetime_part("hour", TimeGenerated)
| project HourOfDay 
| summarize EventCount=count() 
         by HourOfDay
| sort by HourOfDay asc 

```
### Todynamic
Esta función permite convertir un campo que contenga un json en campos seleccionables y operables.
```
SecurityAlert
| extend ExtProps=todynamic(ExtendedProperties) //Columna que almacena el json
| project AlertName 
        , TimeGenerated 
        , AlertStartTime = ExtProps["Alert Start Time (UTC)"]
        , Source = ExtProps.Source
        , NonExistentUsers = ExtProps["Non-Existent Users"]
        , ExistingUsers = ExtProps["Existing Users"]
        , FailedAttempts = ExtProps["Failed Attempts"]
        , SuccessfulLogins = ExtProps["Successful Logins"]
        , SuccessfulUserLogins = ExtProps["Successful User Logons"]
        , AccountLogonIds = ExtProps["Account Logon Ids"]
        , FailedUserLogins = ExtProps["Failed User Logons"]
        , EndTimeUTC = ExtProps["End Time UTC"]
        , ActionTaken = ExtProps.ActionTaken
        , ResourceType = ExtProps.resourceType
        , ServiceId = ExtProps.ServiceId
```
Para realizar esta operación con json de varios niveles se debe usar la notación "ExtProps.Level1.Level2" 

### Iif
Operador lógico "si", sirve para crear campos dinámicos teniendo en cuenta una condición.

```
Perf
| where CounterName == "% Free Space"
| extend FreeState = iif( CounterValue < 50
                        , "You might want to look at this" // Si al condición se cumple
                        , "You're OK!" // Si la condición no se cumple
                        )
| project Computer
        , CounterName
        , CounterValue 
        , FreeState
```
### Case
Operador lógico de un switch, se genera un campo dinámico pudiendo declarar varias opciones.
```
Perf
| where CounterName == "% Free Space"
| extend FreeLevel = case( CounterValue < 10, "Critical (Less than 10% free disk space)" // almacena el valor asignado si se cumple la condición
                         , CounterValue < 30, "Danger (10% to 30% free disk space)"
                         , CounterValue < 50, "Look at it (30% to 50% free disk space)"
                         , "You're OK! (More than 50% free disk space)"
                         )
| summarize ComputerCount=count() 
         by FreeLevel
```
### Isempty e Isnull
La función isempty devolverá Verdadero si el campo asignado está vacío e Isnull si contiene el valor Null.
```
Perf
| where isempty( InstanceName )
| count

Perf
| where isnull( SampleCount )
| count
```
### Split
Esta función divide un campo por el carácter/es asignados.
```
Perf
| take 100                       // done just to give us a small dataset to demo
| project Computer 
        , CounterName 
        , CounterValue 
        , CounterPath  // Ejemplo: "\\AppBE00.NA.contosohotels.com\LogicalDisk(D:)\% Free Space	"
        , CPSplit = split(CounterPath, "\\") // Salida: ["","","AppBE00.NA.contosohotels.com","LogicalDisk(D:)","% Free Space"]	
```
Se puede añadir un tercer parámetro a la función que será el índice del elemento de salida que queremos obtener.
```
Perf
| take 100                       
| extend myComputer = split(CounterPath, "\\", 2) // Salida: ["CH-UBNTVM"]	
       , myObjectInstance = split(CounterPath, "\\", 3)
       , myCounterName = split(CounterPath, "\\", 4)  
| project Computer 
        , ObjectName 
        , CounterName 
        , InstanceName 
        , myComputer 
        , myObjectInstance
```
Es más recomendable asignar la salida del split a una variable y obtener de ahí el índice deseado, la salida será un String no un array.
```
Perf
| extend CounterPathArray = split(CounterPath, "\\") 
| extend myComputer = CounterPathArray[2] // Salida: MABS20.NA.contosohotels.com	
       , myObjectInstance = CounterPathArray[3]
```
### String Operators
Operadores adicionales a los explicados en el primer apartado:
* `where CouenterName contains "BYTES"` No es sensitivo
* `where CouenterName contains_cs "BYTES"` Es sensitivo
* `where CouenterName !contains "BYTES"` Niega el contiene
* `where CounterName in ("Disk Transfers/sec", "Disk Reads/sec", "Avg. Disk sec/Write") ` Pertenece a un grupo

## Agregaciones
* `arg_max(CounterValue, *)` Para devolver el valor más alto de la respuesta.
* `arg_min(CounterValue, *)` Para devolver el valor más bajo de la respuesta.

### Maseket() y makelist()
Crear listas de elementos con el resultado. En el ejemplo, crea una lista de los equipos con menos del 30% de capacidad de disco. 
Makeset crea una lista de elementos únicos y makelist con todos los elementos.
```
  Perf
| where CounterName == "% Free Space"
    and CounterValue <= 30
| summarize Computers = makeset(Computer)
```

### Mxexpand()
Pasa los elementos de una lista a un resultado por columna, repitiendo el resto de los datos en la fila.
```
SecurityAlert
| extend ExtProps=todynamic(ExtendedProperties)
| mvexpand ExtProps
| project TimeGenerated 
        , DisplayName 
        , AlertName 
        , AlertSeverity 
        , ExtProps
```

## Dcount()
Dcount hace una cuenta de elementos únicos de manera aproximada.  
A diferencia de `distinct`, dcount es muy rápido y puede servir para hacer una aproximación.  
```
SecurityEvent
| where TimeGenerated >= ago(90d)
| summarize dcount(EventID) by Computer
| sort by Computer asc
```
Se le puede añadir un parámetro a dcount con el nivel de precisión.
* 0 = Least accurate, 1.6% error
* 1 = Default, balances accuracy and time, 0.8% error level
* 2 = Accurate but slow, 0.4% error
* 3 = Extra accurate but slowest, 0.28% error level

Se puede utilizar la función `dcountif()`, para utilizarle es necesario pasarle un segundo parámetro.  
En el segundo parámetro se añadirá una lista de valores. Al igual que a dcount, se le puede pasar como tercer parámetro la precisión.  

```
SecurityEvent
| where TimeGenerated >= ago(90d)
| summarize dcountif( EventID
                    , EventID in (4625, 4688, 4624, 4672, 4670, 4689, 4634, 4674)
                    ) 
         by Computer
| sort by Computer asc
```
## Pivot()
Esta función permite crear columnas con los valores de los resultados.  
```
Event
| project Computer, EventLevelName 
| evaluate pivot(EventLevelName)
| sort by Computer asc
```

## Datasets
### Let
Let se utiliza para la creación de variables, constantes o funciones.
* Creación de constantes `let minCounterValue = 300;` 
* También puede ser usado con datos calculados como `let startDate = ago(7d)`
* Se pueden almacenar resultados de una consulta.
```
let Updt = Update
| where Computer = "contosoeqp"
| project ...
```  
* Para la creación de funciones.
```
let dateDiffInDays = (date1: datetime, date2: datetime)
                     { (date1-date2) / 1d};
print dateDiffInDays(now(), todatetime("2018-05-01"))
```
### Join
Sirve para unir los datos de dos o más tablas.
```
Perf
| take 10
| join (Alert) on Computer
```
También se podría haber utilizado `| join (Alert) on $left.Computer == $right.Computer`, útil cuando no tienen el mismo nombre.
La consulta antes del join, puede ser todo lo compleja que se necesite, al igual que en el interior del paréntesis posterior al join.
Los tipos de join se mancan con `| join kind=tipo` y existen varios tipos de join:
* innerunique, valor por defecto. Devuelve el primer resultado de la derecha que coincida con cada resultado de la derecha y descarta el resto.
* inner, una fila por cada combinación de resultados. Igual que en SQL.
* leftouter, un resultado por cada columna de la izquierda, incluso aunque no coincida con ningún resultado de la derecha.
* rightouter / fullouter, todos los resultados de la derecha independientemente de no tener coincidencia.
* leftanti / rightanti, sólo devuelve resultados que no tengan coincidencia en la otra tabla.
* leftsemi / rightsemi, devuelve filas que coincidan en ambas, pero únicamente las columnas de una de ellas.

### Union
Sirve para unir dos tablas, no mezcla los elementos en una misma fila, crea filas diferentes para cada elemento.  
En el caso de que una columna no exista en la otra tabla, en la tabla en la que no existe aparecerá sin datos.  
```
UpdateSummary
| union Update

UpdateSummary
| union  withsource="SourceTable" Update //Mostrará una columna nueva llamada "SourceTable" con el nombre de la tabla de cada registro.  
```
Hay dos tipos de "Union",  "inner" solo devuelve las celdas en común.
"outer" por defecto, devuelve todas las celdas quedando vacías las que no coincidan. 
```
UpdateSummary
| union kind=outer Update

```

Mirar "External" si se puede hacer la consulta dinámicamente
