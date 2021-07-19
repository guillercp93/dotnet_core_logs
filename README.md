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