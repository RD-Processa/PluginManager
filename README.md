# PluginManager

Los plugins o complementos son la forma de registrar un script de PowerShell dentro de un repositorio para su posterior uso.

Para registrar un plugin se utiliza la función `Install-Plugin`

Un plugin es un archivo zip que contiene al menos dos archivos:


## Manifiesto 
Es un archivo en formato XML donde se definen las propiedades del plugin como el nombre, número de versión, etc. El nombre de este archivo debe corresponder con el valor del parámetro `ManifestName` al utilizar la función `Install-Plugin`.

### Formato de un archivo de manifiesto:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no" ?>
<!-- File encode must be UTF-8 -->
<promail>
	<plugin>
		<name>MyCoolName</name>
		<description>Lorem ipsum dolor sit amet consectetur adipiscing elit</description>
		<version>01.00.00</version>
		<author>Pepe Cortizona, Ungenio González</author>
		<organization>Processa</organization>
		<file>MyScript.ps1</file>
	</plugin>
	<configuration>
		<requires type="SmtpConnection" withName="MySmtpConnection"/>
	</configuration>	
</promail>
```

Nodo | Descripción
------------ | -------------
promail | Nodo raíz en el archivo XML. Puede utilizar cualquier nombre siempre que corresponda con el valor del parámetro `RootName` en la función `Install-Plugin`.
plugin | Nodo que define las propiedades del plugin.
configuration | Nodo que define las entradas de configuración que deben existir antes de instalar el plugin (opcional).

#### Nodo Plugin

Elemento | Descripción
------------ | -------------
name| Nombre que identifica de forma univoca al plugin. 
description | Texto que describe la funcionalidad del plugin.
version | Número de versión del plugin en formato major.minor[.build[.revision]]
author | Nombre del autor del plugin.
organization | Nombre de la organización que desarrolla el plugin.
file | Nombre del archivo script de PowerShell en el archivo zip. Puede utilizar cualquier nombre de archivo valido, siempre que finalice con la extensión .ps1

#### Nodo configuration
requires | Define una entrada de configuración requerida. (puede agregar tantas como sean necesarias).

#### Elemento requires

Atributo | Descripción
------------ | -------------
type | Puede contener alguno de los siguientes valores: `SmtpConnection`, `FtpConnection`, `SqlConnection`, `RabbitConnection`, `PGPKeyStore` o `AppSetting`. El proceso de instalación validará que exista una entrada local del tipo correspondiente utilizando el módulo `PSProcessa`.
withName | Define el nombre con el que `PSProcessa` busca la entrada de configuración.


El valor de `type` define la función de `PSProcessa` que se utiliza para hacer la validación de la entrada de configuración de la siguiente forma:

Type | PSProcessa function
------------ | -------------
SmtpConnection | `Get-SmtpConnection`
FtpConnection |  `Get-FtpConnection`
SqlConnection | `Get-SqlConnection`
RabbitConnection | `Get-RabbitConnection`
PGPKeyStore | `Get-PGPKeyStore`
AppSetting | `Get-AppSetting`


## Script 
Es un archivo script de PowerShell  con la extensión ps1. Debe contener un bloque de parámetros donde se define al menos uno con el nombre `InputObject` de tipo `PSObject`.

### Formato de un archivo de script:

```powershell
param                                                                                                                                           
(                                                                                                                                               
        [Parameter(Mandatory)]                                                                                                                  
        [PSObject]                                                                                                                              
        $InputObject                                                                                                                            
)                                                                                                                                               
                                                                                                                                                
# Your code from here                                                                                                                           
```

### InputObject
Es un objeto de tipo `System.Management.Automation.PSObject` con las siguientes claves:

Clave | Descripción | Tipo de dato
------------ | ------------- | -------------
Path | Ruta de acceso del archivo adjunto al correo electrónico | `string`
Subject | Asunto del correo electrónico | `string`
DateTime | Fecha y hora en la que se recibió el correo electrónico | `DateTime`
From | Dirección de correo del remitente del correo electrónico | `string`
CorrelationalId | Valor de identificador único adjunto al correo electrónico que permite la referencia a una transacción particular | `string`

El proceso de instalación de un plugin validará la existencia de estos dos archivos en el archivo zip, validará el formato del manifiesto contra el esquema XSD que se especifique en el parámetro SchemaPath de la función Install-Plugin y copiará el archivo script a una ruta local para su posterior uso. 

Para conocer los plugin registrados puede utilizar la función `Get-Plugin`.

`Get-Plugin` admite el uso del parámetro `Name`, así que para obtener la ruta de acceso de un script registrado previamente con `Install-Plugin`, bastaría con conocer el nombre del plugin y utilizarlo como parámetro en `Get-Plugin`. El objeto de retorno tendrá una propiedad con el nombre `FilePath` que corresponde con la ruta de acceso del archivo script de PowerShell. 

```powershell
(Get-Plugin -Name 'MyName').FilePath
```

