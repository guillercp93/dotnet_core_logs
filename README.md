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

## Paso 3: Configuración Salida a Archivo

Para este caso, se agregará otro ítem a "WriteTo" con la configuración del archivo de salida para los logs.

    {
    	"Name": "File",
    	"Args": {
	    	"path": "path/to/file.txt",
	    	"fileSizeLimitBytes": 1000000,
	    	"rollOnFileSizeLimit": true,
	    	"flushToDiskInterval": 1
	    }
	}
En la configuración que se vió anterior se puso la siguiente configuración:

 - **path**: Ubicación del archivo donde se escribirán los logs.
 - **fileSizeLimitBytes**: Tamaño máximo que tendrá el archivo. En este caso es 1GB.
 - **rollOnFileSizeLimit**: Especifica que cuando se llegue al tamaño máximo se creará otro archivo de log usando una secuencia.
 - **flushToDiskInterval**: Periodo de tiempo en que se irán borrando los logs más viejos.

Para más información sobre [Serilog.Sinks.File](https://github.com/serilog/serilog-sinks-file)

## Paso 4: Configuración Salida a Base de Datos.

Para este caso, la base de datos que se va a usar es PostgresSql, pero Serilog tiene salida para otros motores de base de datos como Sql Server, MariaDB, MongoDB, SqLite, etc.
Según [Serilog.Sinks.PostgreSQL](https://github.com/b00ted/serilog-sinks-postgresql), la configuración base, recae sobre la sección "WriteTo":

    "WriteTo": [
          {
            "Name": "PostgreSQL",
            "Args": {
              "connectionString": "",
              "tableName": "",
              "needAutoCreateTable": true
            }
          }
        ]
Los argumentos que recibe son:

 - **connectionString**: La cadena de conexión a la base de datos
 - **tableName**: Nombre de la tabla donde se registrarán los logs
 - **needAutoCreateTable**: si es verdadero, creará una nueva tabla con el nombre configurado en tableName con la siguiente estructura:
	 - **message(text):** Contiene el mensaje del log
	 - **template_message(text):** Contiene el mensaje sin interpolar variables
	 - **level (int):** Nivel del log
	 - **timestamp(timestamp):** Fecha de registro del log
	 - **exception(text):** Mensaje de la excepción lanzada
	 - **log_event(json):** Objeto JSON con los valores de las columnas anteriores.

Para la modificación o creación de las columnas que llevará la tabla, se debe instalar para este caso **Serilog.Sinks.PostgreSQL.Configuration**, luego registrarlo en la lista "Using" dentro de la llave "Serilog"

    "Serilog": {
	   "Using": [
		   "Serilog.Sinks.PostgreSQL.Configuration"
	   ],
    }

Luego creamos una llave a **nivel 1** de todo el json de **appsettings**, con el valor "Columns". Allí contendrá un objeto donde cada llave será el nombre de la columna y el valor podrá ser el tipo por defecto con el que Serilog deberá llenar esa columna.

    "Columns": {
        "message": "RenderedMessageColumnWriter",
        "message_template": "MessageTemplateColumnWriter",
        "level": {
          "Name": "LevelColumnWriter",
          "Args": {
            "renderAsText": true,
            "dbType": "Varchar"
          }
        },
        "raise_date": "TimestampColumnWriter",
        "exception": "ExceptionColumnWriter",
        "properties": "LogEventSerializedColumnWriter",
        "props_test": {
          "Name": "PropertiesColumnWriter",
          "Args": { "dbType": "Json" }
        },
        "machine_name": {
          "Name": "SinglePropertyColumnWriter",
          "Args": {
            "propertyName": "MachineName",
            "writeMethod": "Raw"
          }
        }
      }