## Paso 1: Instalaci�n y Configuraci�n

Instalar las librer�as:

 - Serilog (`dotnet add package Serilog.AspNetCore`)
 - Lectura del archivo appsetting.json (`dotnet add package Serilog.Settings.Configuration`)
 - Salida a consola (`dotnet add package Serilog.Sinks.Console`)
 - Salida a un archivo (`dotnet add package Serilog.Sinks.File`)
 - Salida a base de datos (`dotnet add package Serilog.Sinks.PostgreSQL`)

Para m�s informaci�n de los Sinks o salidas que ofrece Serilog puede ir a la p�gina [Provided Sinks](https://github.com/serilog/serilog/wiki/Provided-Sinks)

Configuramos Serilog en el archivo *Program.cs* para que nuestra aplicaci�n use la librer�a.
Para ello, importamos serilog en el archivo Program.cs

    using  Serilog;

Luego creamos una variable est�tica en el cual leeremos el archivo de configuraci�n de la aplicaci�n seg�n el ambiente de ejecuci�n.

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
En el m�todo Main, se crea un objeto **LoggerConfiguration** donde le decimos a Serilog que use la configuraci�n que se ley� en el paso anterior.

    Log.Logger = new LoggerConfiguration()
				    .ReadFrom.Configuration(Configuration)
				    .CreateLogger();
Por �ltimo, le decimos al host que debe usar el **middleware** de Serilog.

    Host.CreateDefaultBuilder(args)
	    .UseSerilog()
	    .ConfigureWebHostDefaults(webBuilder => 
	    {
		    webBuilder.UseStartup<Startup>();
		    webBuilder.UseSerilog();
		    webBuilder.ConfigureKestrel(options => options.AllowSynchronousIO = true);
		});
