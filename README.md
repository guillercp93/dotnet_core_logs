## Paso 4: Configuraci�n Salida a Base de Datos.

Para este caso, la base de datos que se va a usar es PostgresSql, pero Serilog tiene salida para otros motores de base de datos como Sql Server, MariaDB, MongoDB, SqLite, etc.
Seg�n [Serilog.Sinks.PostgreSQL](https://github.com/b00ted/serilog-sinks-postgresql), la configuraci�n base, recae sobre la secci�n "WriteTo":

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

 - **connectionString**: La cadena de conexi�n a la base de datos
 - **tableName**: Nombre de la tabla donde se registrar�n los logs
 - **needAutoCreateTable**: si es verdadero, crear� una nueva tabla con el nombre configurado en tableName con la siguiente estructura:
	 - **message(text):** Contiene el mensaje del log
	 - **template_message(text):** Contiene el mensaje sin interpolar variables
	 - **level (int):** Nivel del log
	 - **timestamp(timestamp):** Fecha de registro del log
	 - **exception(text):** Mensaje de la excepci�n lanzada
	 - **log_event(json):** Objeto JSON con los valores de las columnas anteriores.

Para la modificaci�n o creaci�n de las columnas que llevar� la tabla, se debe instalar para este caso **Serilog.Sinks.PostgreSQL.Configuration**, luego registrarlo en la lista "Using" dentro de la llave "Serilog"

    "Serilog": {
	   "Using": [
		   "Serilog.Sinks.PostgreSQL.Configuration"
	   ],
    }

Luego creamos una llave a **nivel 1** de todo el json de **appsettings**, con el valor "Columns". All� contendr� un objeto donde cada llave ser� el nombre de la columna y el valor podr� ser el tipo por defecto con el que Serilog deber� llenar esa columna.

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