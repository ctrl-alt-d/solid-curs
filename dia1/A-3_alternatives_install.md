# Instal·lar dotnet

Diverses maneres altenatives per instal·lar dotnet si no tens permisos admin a la màquina,

---

## Dotnet SDK (sense permisos admin I)
### Descarregar binaris

* Navega fins a [Download .NET](https://dotnet.microsoft.com/en-us/download) i clica a All .NET 10.0 downloads.
* Descarrega des de "Windows binaries" la versió per al teu sistema operatiu, segurament [x64](https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/sdk-10.0.300-windows-x64-binaries)
* Descomprimeix, canvia el nom a la carpeta principal a `dotnet` i mou la carpeta a la teva carpeta de treball (ex: `c:\users\dherrera`)

---

## Dotnet SDK (sense permisos admin I)
### Variables d'entorn

* Afegeix al path: `[Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";$env:USERPROFILE\dotnet", "User")`
* Afegeix variable dotnet root: `[Environment]::SetEnvironmentVariable("DOTNET_ROOT", "$env:USERPROFILE\dotnet", "User")`
* Obre una nova power shell i compriva que l'sdk està disponible: `dotnet --list-sdks`


---

## Dotnet SDK (sesne permisos admin II amb binaris)


```powershell
# Instal·la .NET 10 SDK x64 sense permisos d'administrador
$Version = "10.0.300"
$ZipFile = "$env:TEMP\dotnet-sdk.zip"
$InstallDir = "$env:USERPROFILE\dotnet"

$Url = "https://builds.dotnet.microsoft.com/dotnet/Sdk/$Version/dotnet-sdk-$Version-win-x64.zip"
Invoke-WebRequest -Uri $Url -OutFile $ZipFile
Expand-Archive -Path $ZipFile -DestinationPath $InstallDir
[Environment]::SetEnvironmentVariable("PATH","$UserPath;$InstallDir","User")
[Environment]::SetEnvironmentVariable("DOTNET_ROOT",$InstallDir,"User")
```
