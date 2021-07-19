## Paso 3: Configuraci�n Salida a Archivo

Para este caso, se agregar� otro �tem a "WriteTo" con la configuraci�n del archivo de salida para los logs.

    {
    	"Name": "File",
    	"Args": {
	    	"path": "path/to/file.txt",
	    	"fileSizeLimitBytes": 1000000,
	    	"rollOnFileSizeLimit": true,
	    	"flushToDiskInterval": 1
	    }
	}
En la configuraci�n que se vi� anterior se puso la siguiente configuraci�n:

 - **path**: Ubicaci�n del archivo donde se escribir�n los logs.
 - **fileSizeLimitBytes**: Tama�o m�ximo que tendr� el archivo. En este caso es 1GB.
 - **rollOnFileSizeLimit**: Especifica que cuando se llegue al tama�o m�ximo se crear� otro archivo de log usando una secuencia.
 - **flushToDiskInterval**: Periodo de tiempo en que se ir�n borrando los logs m�s viejos.

Para m�s informaci�n sobre [Serilog.Sinks.File](https://github.com/serilog/serilog-sinks-file)