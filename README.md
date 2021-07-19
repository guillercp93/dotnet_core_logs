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