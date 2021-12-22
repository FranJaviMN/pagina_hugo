---
title: "Scripts y Automatización en Windows"
date: 2020-12-21T13:18:45+01:00
draft: true
---
# Scripts en Windows ¿Que es shell script?

Uno de los componentes software más importantes e incrustados en un sistema operativo, es la terminal o consola. Un tipo de maquinaria central donde se pueden ejecutar instrucciones que el sistema entiende y puede realizar.

Todas estas instrucciones que se le pasan al sistema por medio de comandos, puedes agruparlas en un fichero o archivo, permitiendo así, ejecutar un conjunto de órdenes para que las realice paso a paso de forma procedural. Esto brinda cierta automatización sobre tareas.

Todo aquel dispositivo informático que tenga un sistema operativo, es bien visto de tener una terminal, bien puede ser de los más conocidos y usados como los de las familias Apple o GNU/Linux con su intérprete de comandos Shell Script Bash, del mismo modo, los sistemas operativos de la familia Microsoft Windows, tienen su propio intérprete de comandos llamado Shell Script Batch.

## ¿Que es powershell?

**PowerShell** es una interfaz de consola **(CLI)** con posibilidad de escritura y unión de comandos por medio de instrucciones (scripts). Esta interfaz de consola está diseñada para su uso por parte de **administradores de sistemas**, con el propósito de automatizar tareas o realizarlas de forma más controlada. Originalmente denominada como MONAD en 2003, su nombre oficial cambió al actual cuando fue lanzada al público el 25 de abril de 2006. El 15 de agosto de 2016, Microsoft publicó el código fuente de PowerShell en GitHub, y cambió su nombre a PowerShell Core.

### Conceptos básicos

Debemos de tener en cuenta que, si ya hemos usado script en Bash este va a ser muy parecido a como se hace en Windows. A continuación vamos a ver algunos conceptos basicos a la hora de crear scripts en nuestra powershell:

* PowerShell y C# comparten parte de su sintaxis: Esto le brinda un gran valos a la hora de ser migrado de powershell a C# o viceversa, lo que lo comvierte en una utilidad muy importante.

* La sintaxis de los comandos es similar a otras shells: El comando empieza por el nombre de ese comando, seguido de parametros y, a su vez, los argumentos de esos parametros:
```powershell
Por ejemplo
PS C:\Users\Francisco Javier> write-output -InputObject Hola
Hola
```

* Algunas caracteristicas de powershell es que no diferencia entre mayusculas y minusculas, como en la shell de Linux, por lo que podemos escribir los comandos con mayuscula o minusculas. Tambien tenemos un parametro especial **--** que hace que, cualquier cadena que vaya detras de el en un comando sera considerado como un argumento:
```powershell
PS C:\Users\Francisco Javier> write-output -- Hola
Hola
```

### Categorias de comandos.
* **Un cmdlet** es un script ligero de Windows PowerShell que realiza una sola función. Cada cmdlet tiene un archivo de ayuda al que se puede acceder escribiendo **Get-Help <cmdlet-Name> -Detailed**. La vista detallada del archivo de ayuda de cmdlet incluye una descripción del cmdlet, la sintaxis del comando, descripciones de los parámetros y un ejemplo que demuestra el uso del cmdlet. Aunque si solo queremos ver lo basico podemos usar **<cmdlet-Name> -?**
    ```powershell
    PS C:\Users\Francisco Javier> Get-help Copy-Item -detailed
    NOMBRE
    Copy-Item

    SINTAXIS
    Copy-Item [-Path] <string[]> [[-Destination] <string>]  [<CommonParameters>]

    Copy-Item [[-Destination] <string>]  [<CommonParameters>]

    PARÁMETROS
    -Confirm

    -Container

    -Credential <pscredential>

    -Destination <string>

    -Exclude <string[]>

    -Filter <string>

    -Force

    -FromSession <PSSession>

    -Include <string[]>

    -LiteralPath <string[]>

    -PassThru

    -Path <string[]>

    -Recurse

    -ToSession <PSSession>

    -UseTransaction

    -WhatIf

    <CommonParameters>
        Este cmdlet admite los parámetros comunes: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable y OutVariable. Para obtener más información, consulta
        about_CommonParameters (https:/go.microsoft.com/fwlink/?LinkID=113216).


    ALIAS
    cpi
    cp
    copy
    ```
