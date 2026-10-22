# SS-Check - Revisión de FiveM consentida por el jugador
# Solo lista nombres de procesos y archivos que coinciden con palabras clave.
# No lee el contenido de archivos, no sube nada, no se instala.

$keywords = @(
 'eulen','redengine','red engine','skript','lynx','desudo','tzx','hxcheat',
 'kiddion','cherax','2take1','impulse','phantom','midnight','executor',
 'injector','aimbot','triggerbot','wallhack','esp','modmenu','spoofer',
 'cheatengine','cheat engine','extreme injector','processhacker'
)

Clear-Host
Write-Host "=== SS CHECK FIVEM ===" -ForegroundColor Cyan
Write-Host @"
Este programa va a:
 - Listar procesos en ejecución y títulos de ventana
 - Listar archivos de la carpeta plugins/mods de FiveM
 - Buscar NOMBRES de archivos sospechosos en Descargas, Escritorio, Recientes y Prefetch
No lee el contenido de tus archivos ni envía datos a ningún sitio.
"@
$ok = Read-Host "Escribe SI para continuar"
if ($ok -ne 'SI') { exit }

$report = @()
function Match($text) { foreach ($k in $keywords) { if ($text -like "*$k*") { return $k } } return $null }
function Section($t) { Write-Host "`n--- $t ---" -ForegroundColor Yellow; $script:report += "`n--- $t ---" }
function Hit($msg)   { Write-Host "[!] $msg" -ForegroundColor Red; $script:report += "[!] $msg" }
function Info($msg)  { Write-Host "    $msg"; $script:report += "    $msg" }

# 1. Procesos
Section "Procesos sospechosos"
$found = $false
Get-Process | ForEach-Object {
    $k = Match ($_.ProcessName + ' ' + $_.MainWindowTitle)
    if ($k) { Hit "$($_.ProcessName) | '$($_.MainWindowTitle)' (coincide: $k)"; $found = $true }
}
if (-not $found) { Info "Sin coincidencias" }

# 2. Carpetas de FiveM
Section "FiveM plugins / mods"
$fivem = "$env:LOCALAPPDATA\FiveM\FiveM.app"
foreach ($sub in 'plugins','mods') {
    $p = Join-Path $fivem $sub
    if (Test-Path $p) {
        $items = Get-ChildItem $p -Recurse -ErrorAction SilentlyContinue
        if ($items) { $items | ForEach-Object { Info "$sub\$($_.Name)  ($($_.LastWriteTime))" } }
        else { Info "$sub vacío" }
    } else { Info "$sub no existe" }
}

# 3. Archivos por nombre
$paths = @{
  'Descargas' = "$env:USERPROFILE\Downloads"
  'Escritorio' = [Environment]::GetFolderPath('Desktop')
  'Recientes'  = "$env:APPDATA\Microsoft\Windows\Recent"
  'Prefetch'   = "C:\Windows\Prefetch"
}
foreach ($name in $paths.Keys) {
    Section "Nombres en $name"
    $p = $paths[$name]
    if (-not (Test-Path $p)) { Info "No accesible (¿falta ejecutar como administrador?)"; continue }
    $found = $false
    Get-ChildItem $p -Recurse -ErrorAction SilentlyContinue | ForEach-Object {
        $k = Match $_.Name
        if ($k) { Hit "$($_.Name) (coincide: $k) - $($_.LastWriteTime)"; $found = $true }
    }
    if (-not $found) { Info "Sin coincidencias" }
}

Section "FIN"
$out = "$([Environment]::GetFolderPath('Desktop'))\SS-report.txt"
$report | Out-File $out -Encoding utf8
Write-Host "`nInforme guardado en: $out" -ForegroundColor Green
Read-Host "Pulsa Enter para salir"
