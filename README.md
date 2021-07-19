## Paso 2: Configuración Salida a Consola

Con el paso anterior, automatizamos la forma en que podemos configurar cada una de las salidas que queramos con Serilog desde el archivo de configuración de la aplicación: **appsettings.json**.
Ahora haremos la configuración de la salida a la consola de comandos o terminal. Abriremos el archivo appsettings.json y agregamos la llave con el nombre "Serilog", el cuál contiene otras llaves "Using", "MinimumLevel", "WriteTo".

      "Serilog": {
        "Using": [],
        "MinimumLevel": {},
        "WriteTo": []
      }
En la llave "Using", listaremos los **Sinks** o destinos que estaremos usando para escribir los logs. En este caso colocaremos el de consola **Serilog.Sinks.Console**.
En la llave "MinimumLevel", sobreescribiremos los [niveles de información de los Logs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-5.0#log-level)  que queramos registrar.

    "MinimumLevel": {
          "Default": "Debug",
          "Override": {
            "Microsoft": "Error",
            "Microsoft.Hosting.Lifetime": "Warning",
            "System": "Warning"
          }
        },
Por último en la llave "WriteTo", configuraremos la salida a consola. En ella podemos modificar la plantilla con la que se mostrarán los logs en la consola y se puede personalizar con temas coloridos o formatos más enriquecidos.
Para más información de [Serilog.Sinks.Console](https://github.com/serilog/serilog-sinks-console)

    "WriteTo": [
    {
	    "Name": "Console",
	    "outputTemplate": "[{Timestamp:HH:mm:ss} {Level}] {SourceContext}{NewLine}{Message:lj}{NewLine}{Exception}{NewLine}",
	    "theme": "Serilog.Sinks.Console"
    },
