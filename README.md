# Manejo de Logs con Serilog en dotnet Core!

Se hará una aplicación en .net (dotnet) Core para explicar el manejo de los logs usando la librería de **serilog**. Para el ejemplo escribiremos los logs que genera una aplicación web en la *consola*, en un *archivo* y en una *base de datos*.


## Paso 1: Instalación y Configuración

Instalar las librerías:

 - Serilog (`dotnet add package Serilog.AspNetCore`)
 - Lectura del archivo appsetting.json (`dotnet add package Serilog.Settings.Configuration`)
 - Salida a consola (`dotnet add package Serilog.Sinks.Console`)
 - Salida a un archivo (`dotnet add package Serilog.Sinks.File`)
 - Salida a base de datos (`dotnet add package Serilog.Sinks.PostgreSQL`)

Para más información de los Sinks o salidas que ofrece Serilog puede ir a la página [Provided Sinks](https://github.com/serilog/serilog/wiki/Provided-Sinks)

Configuramos Serilog en el archivo *Program.cs* para que nuestra aplicación use la librería.
Para ello, importamos serilog en el archivo Program.cs

    using  Serilog;

Luego creamos una variable estática en el cual leeremos el archivo de configuración de la aplicación según el ambiente de ejecución.

    private static IConfiguration Configuration
    {
	    get {
		    string env = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Production";
		    return new ConfigurationBuilder()
					    .SetBasePath(System.IO.Directory.GetCurrentDirectory())
						.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
						.AddJsonFile($"appsettings.{env}.json", optional: true, reloadOnChange: true)
						.AddEnvironmentVariables()
						.Build();
		    }
    }
En el método Main, se crea un objeto **LoggerConfiguration** donde le decimos a Serilog que use la configuración que se leyó en el paso anterior.

    Log.Logger = new LoggerConfiguration()
				    .ReadFrom.Configuration(Configuration)
				    .CreateLogger();
Por último, le decimos al host que debe usar el **middleware** de Serilog.

    Host.CreateDefaultBuilder(args)
	    .UseSerilog()
	    .ConfigureWebHostDefaults(webBuilder => 
	    {
		    webBuilder.UseStartup<Startup>();
		    webBuilder.UseSerilog();
		    webBuilder.ConfigureKestrel(options => options.AllowSynchronousIO = true);
		});

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
