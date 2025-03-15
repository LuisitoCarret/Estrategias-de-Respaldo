# Estrategias-de-Respaldo

***1. Introduccion***

En el siguiente documento se describe el diseño y funcionalidad del procedimiento almacenado encargado de realizar respaldos completos(Full Backup) de
bases de datos en SQL Server.Ademas se incluyen validaciones para evitar errores comunes y garantizar la integridad de los datos.Cabe destacar que posteriormente
ajustaremos el store procedure para respaldar mas tipos de copias de seguridad.

***2. Descripcion del procedimiento***

El procedimiento almacenado permite generar copias de seguridad completas de una base de datos especifica.Los respaldos se guardaran en la ubicacion que se eliga y el nombre del archvio incluira la fecha para facilitar su identificacion.

```SQL
ALTER PROCEDURE sp_BackupFull
    @DatabaseName NVARCHAR(128),
    @PathBackup NVARCHAR(255)
AS
BEGIN
    SET NOCOUNT ON;

    -- Validar si la base de datos existe
    IF NOT EXISTS (SELECT 1 FROM sys.databases WHERE name = @DatabaseName)
    BEGIN
        PRINT 'Error: La base de datos especificada no existe.';
        RETURN;
    END;

    DECLARE @SQL NVARCHAR(MAX);
	DECLARE @Fecha NVARCHAR(10);
	DECLARE @BackupFile NVARCHAR(500);

    -- Generar la fecha en formato YYYYMMDD_HHMMSS
    SET @Fecha = FORMAT(GETDATE(), 'yyyyMMdd');

    -- Construir el nombre del archivo de backup
    SET @BackupFile = @PathBackup + '\' +@DatabaseName + '_Full_' + @Fecha + '.bak';

    -- Intentar ejecutar el backup
    BEGIN TRY
        SET @SQL = 'BACKUP DATABASE [' + @DatabaseName + '] 
                    TO DISK = ''' + @BackupFile + '''
                    WITH FORMAT, INIT, NAME = ''' + @DatabaseName + '_Full Backup''';

        EXEC sp_executesql @SQL;

        PRINT 'Backup Full realizado correctamente en: ' + @BackupFile;
    END TRY
    BEGIN CATCH
        PRINT 'Error al realizar el backup: ' + ERROR_MESSAGE();
    END CATCH;
END;
GO
```

***3. Parametros de entrada***

* @DatabaseName, nombre de la base de datos a respaldar
* @PathBackup, ruta donde se alamcenara el archivo de respaldo

***4. Validaciones implementadas***
Para garantizar el correcto funcionamiento del procedimiento almacenado, se aplican las siguientes validaciones:

**4.1 Verificacion de la existencia de una base de datos**

 * Problema: Intentar realizar un respaldo de una base de datos no existente generara un error
 * Solucion Se consulta sys.databases para verificar si la base de datos existe antes de proceder

```SQL
--- Validar si la base de datos existe
    IF NOT EXISTS (SELECT 1 FROM sys.databases WHERE name = @DatabaseName)
```

**4.2 Validacion del nombre del archivo de respaldo**
 * Problema: Archivos con nombres iguales
 * Solucion: El nombre del archvio de respaldo incluira la fecha para evitar sobreescrituras accidentales

```SQL
 --- Generar la fecha en formato YYYYMMDD_HHMMSS
    SET @Fecha = FORMAT(GETDATE(), 'yyyyMMdd');

    --- Construir el nombre del archivo de backup
    SET @BackupFile = @PathBackup + '\' +@DatabaseName + '_Full_' + @Fecha + '.bak';
```

**5. Consideraciones adicionales**
 * Se recomienda ejecutar el procedimiento con un usuarioque tenga permisos adecuados en SQL Server
 * Es posible extender el procedimiento para permitir respaldos diferenciales o incrementales segun las necesidades futuras

**6. Conclusión**
Este procedimiento almacenado facilita la creacion de respaldos completos en SQL Server con validaciones robustas para evitar errores.
Su implementacion ayuda a manetenr la seguridad y disponibilidad de los datos.
