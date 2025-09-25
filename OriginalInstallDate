Clear-Host

# =========================
# Original Install Date
# =========================
$installDateRaw = (systeminfo | find "Original Install Date").Split(":",2)[1].Trim()
$installDate    = [datetime]::Parse($installDateRaw)

# =========================
# Event Logs
# =========================
$dmEvent     = Get-WinEvent -LogName "Microsoft-Windows-DeviceManagement-Enterprise-Diagnostics-Provider/Admin" -Oldest -MaxEvents 1 | Select-Object -ExpandProperty TimeCreated
$ntfsEvent   = Get-WinEvent -LogName "Microsoft-Windows-Ntfs/Operational" -Oldest -MaxEvents 1 | Select-Object -ExpandProperty TimeCreated
$systemEvent = Get-WinEvent -LogName "System" -Oldest -MaxEvents 1 | Select-Object -ExpandProperty TimeCreated

$eventLogs = @(
    [PSCustomObject]@{ Name="DeviceManagement-EDP"; Time=$dmEvent },
    [PSCustomObject]@{ Name="NTFS/Operational"; Time=$ntfsEvent },
    [PSCustomObject]@{ Name="System"; Time=$systemEvent }
)

# =========================
# Core DLLs
# =========================
$dllPaths = @(
    "C:\Windows\System32\kernel32.dll",
    "C:\Windows\System32\ntdll.dll",
    "C:\Windows\System32\user32.dll",
    "C:\Windows\System32\gdi32.dll",
    "C:\Windows\System32\advapi32.dll",
    "C:\Windows\System32\shell32.dll",
    "C:\Windows\System32\combase.dll",
    "C:\Windows\System32\sechost.dll",
    "C:\Windows\System32\win32kbase.sys",
    "C:\Windows\System32\win32kfull.sys",
    "C:\Windows\System32\crypt32.dll",
    "C:\Windows\System32\msvcrt.dll",
    "C:\Windows\System32\shcore.dll",
    "C:\Windows\System32\kernelbase.dll"
)

$dllTimes = foreach ($dll in $dllPaths) {
    if (Test-Path $dll) {
        [PSCustomObject]@{
            Name = $dll
            Time = (Get-Item $dll).CreationTime
            Diff = [math]::Abs(($installDate - (Get-Item $dll).CreationTime).TotalMinutes)
        }
    }
}

# =========================
# Display Results
# =========================
Write-Host "=============================="
Write-Host "     Original Install Date     " -ForegroundColor Cyan
Write-Host "=============================="
Write-Host "$installDate`n" -ForegroundColor Cyan

# Event Logs
Write-Host "=============================="
Write-Host "        Event Logs            " -ForegroundColor Yellow
Write-Host "=============================="
foreach ($e in $eventLogs) {
    $diff = if ($e.Time) { [math]::Abs(($installDate - $e.Time).TotalMinutes) } else { "N/A" }
    Write-Host ("{0,-35} {1,-25} Δ {2} min" -f $e.Name, $e.Time, $diff) -ForegroundColor Cyan
}

# DLLs
Write-Host "`n=============================="
Write-Host "       Core System DLLs        " -ForegroundColor Yellow
Write-Host "=============================="
foreach ($d in $dllTimes | Sort-Object Diff) {
    Write-Host ("{0,-40} {1,-25} Δ {2} min" -f $d.Name, $d.Time, [math]::Round($d.Diff,2)) -ForegroundColor Cyan
}

# =========================
# Validation
# =========================
$maxEventDiff = $eventLogs | ForEach-Object { if ($_.Time) { [math]::Abs(($installDate - $_.Time).TotalMinutes) } } | Measure-Object -Maximum | Select-Object -ExpandProperty Maximum
$maxDllDiff   = $dllTimes | ForEach-Object { $_.Diff } | Measure-Object -Maximum | Select-Object -ExpandProperty Maximum

$eventValid = $maxEventDiff -le 5       # 5 minutes tolerance for logs
$dllValid   = $maxDllDiff -le 100      # 100+ minutes tolerance for DLLs

# =========================
# Display Validation
# =========================
Write-Host ""
Write-Host "=============================="
Write-Host "          VALIDATION           " -ForegroundColor Yellow
Write-Host "=============================="

# Event Logs Validation
if ($eventValid) {
    Write-Host "Event Logs: VALID" -ForegroundColor Green -BackgroundColor Black
} else {
    Write-Host "Event Logs: FAILED TAMPERED" -ForegroundColor Red -BackgroundColor Black
}

# DLLs Validation
if ($dllValid) {
    Write-Host "DLLs: VALID" -ForegroundColor Green -BackgroundColor Black
} else {
    Write-Host "DLLs: FAILED TAMPERED" -ForegroundColor Red -BackgroundColor Black
}
