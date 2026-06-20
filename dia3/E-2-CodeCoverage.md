# Cobertura de codi a .NET

Aquest resum assumeix que ja tens un projecte de tests .NET funcionant.

## Opció 1: amb Visual Studio Code

Si tens l'extensió de C# i els tests detectats, pots executar **Run Tests with Coverage** des del panell de Testing (clic dret sobre els testos).

Això recull cobertura automàticament i és la via més còmoda per al dia a dia.

## Opció 2: des de terminal (sense VS Code)

### 1) Generar dades de cobertura

```bash
dotnet test --collect:"XPlat Code Coverage"
```

El fitxer de cobertura es genera normalment dins de `TestResults/.../coverage.cobertura.xml`.

### 2) Instal·lar ReportGenerator

#### macOS / Linux

```bash
dotnet tool install -g dotnet-reportgenerator-globaltool
```

#### Windows (PowerShell)

```powershell
dotnet tool install -g dotnet-reportgenerator-globaltool
```

Si ja el tens instal·lat i vols actualitzar-lo:

```bash
dotnet tool update -g dotnet-reportgenerator-globaltool
```

### 3) Generar informe HTML

#### macOS / Linux

```bash
reportgenerator \
	-reports:"**/coverage.cobertura.xml" \
	-targetdir:"coveragereport" \
	-reporttypes:Html
```

#### Windows (PowerShell)

```powershell
reportgenerator `
	-reports:"**/coverage.cobertura.xml" `
	-targetdir:"coveragereport" `
	-reporttypes:Html
```

Obre després `coveragereport/index.html` al navegador.

## Troubleshooting

### `reportgenerator` command not found

1. Verifica que l'eina està instal·lada:

```bash
dotnet tool list -g
```

2. Si no surt, instal·la-la:

```bash
dotnet tool install -g dotnet-reportgenerator-globaltool
```

3. Si surt però no es troba la comanda, normalment falta el directori d'eines globals al `PATH`.

Comprova on és l'eina:

```bash
dotnet tool list -g
dotnet tool run reportgenerator --help
```

Si amb `dotnet tool run` funciona, el problema és només de `PATH`.

- macOS / Linux (`zsh` o `bash`): afegeix `~/.dotnet/tools`

```bash
echo 'export PATH="$PATH:$HOME/.dotnet/tools"' >> ~/.zshrc
source ~/.zshrc
```

(Si fas servir `bash`, fes-ho a `~/.bashrc` o `~/.bash_profile`.)

- Windows (PowerShell): afegeix `%USERPROFILE%\\.dotnet\\tools` al `PATH` d'usuari

```powershell
$tools = "$env:USERPROFILE\\.dotnet\\tools"
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$tools", "User")
```

Tanca i obre terminal després del canvi.

### Coverage file not found

1. Assegura't d'haver executat:

```bash
dotnet test --collect:"XPlat Code Coverage"
```

2. Busca el fitxer real generat:

```bash
find . -name "coverage.cobertura.xml"
```

3. Si cal, passa el path exacte a `-reports:` en lloc del patró `**/coverage.cobertura.xml`.
