During a live investigation look at processes via:
* Get-Process
  * Enrich with Get-CimInstance (get parent process and command line)
```
# Get current date, time, and hostname for filename
$timestamp = Get-Date -Format "yyyy-MM-dd_HH-mm-ss"
$hostname = $env:COMPUTERNAME
$txtFilename = "ProcessList_$hostname`_$timestamp.txt"
$csvFilename = "ProcessList_$hostname`_$timestamp.csv"

# Set output paths to the user's desktop
$desktopPath = [System.Environment]::GetFolderPath("Desktop")
$txtOutputPath = [System.IO.Path]::Combine($desktopPath, $txtFilename)
$csvOutputPath = [System.IO.Path]::Combine($desktopPath, $csvFilename)

# Gather all process data first
$allProcesses = Get-CimInstance Win32_Process

# Create a lookup table for Parent Process details
$parentProcessLookup = @{}
$allProcesses | ForEach-Object {
    $parentProcessLookup[$_.ProcessId] = $_
}

# Gather process information
$processes = $allProcesses | ForEach-Object {
    $processName = (Get-Process -Id $_.ProcessId -ErrorAction SilentlyContinue).ProcessName
    $parentProcess = $parentProcessLookup[$_.ParentProcessId]

    [PSCustomObject]@{
        ParentProcessId         = $_.ParentProcessId
        ParentProcessName       = $parentProcess.ProcessName
        ParentProcessCommandLine = $parentProcess.CommandLine
        ProcessId               = $_.ProcessId
        ProcessName             = $processName
        CommandLine             = $_.CommandLine
    }
}

# Export to a neatly formatted text file (list format)
$processes | ForEach-Object {
    @"
=======================================

Parent Process ID        : $($_.ParentProcessId)
Parent Process Name      : $($_.ParentProcessName)
Parent Process Command   : $($_.ParentProcessCommandLine)

Process ID               : $($_.ProcessId)
Process Name             : $($_.ProcessName)
Process Command Line     : $($_.CommandLine)

=======================================
"@
} | Out-File -FilePath $txtOutputPath -Width 500

# Export to CSV for structured analysis
$processes | Export-Csv -Path $csvOutputPath -NoTypeInformation

# Confirm file creation
Write-Host "Process list saved to:"
Write-Host "  - Text file: $txtOutputPath"
Write-Host "  - CSV file: $csvOutputPath"
```

During a live investigation also look at current connections:
* Get-NetTCPConnection
  * Enrich with Get-Process
```
# Get current date, time, and hostname for filename
$timestamp = Get-Date -Format "yyyy-MM-dd_HH-mm-ss"
$hostname = $env:COMPUTERNAME
$txtFilename = "NetConnections_$hostname`_$timestamp.txt"
$csvFilename = "NetConnections_$hostname`_$timestamp.csv"

# Set output paths to the user's desktop
$desktopPath = [System.Environment]::GetFolderPath("Desktop")
$txtOutputPath = [System.IO.Path]::Combine($desktopPath, $txtFilename)
$csvOutputPath = [System.IO.Path]::Combine($desktopPath, $csvFilename)

# Gather all network connections
$connections = Get-NetTCPConnection | ForEach-Object {
    $process = Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue

    [PSCustomObject]@{
        LocalAddress   = $_.LocalAddress
        LocalPort      = $_.LocalPort
        State          = $_.State
        OwningProcess  = $_.OwningProcess
        ProcessName    = $process.ProcessName
        ProcessPath    = $process.Path
    }
}

# Export to a neatly formatted text file (list format)
$connections | ForEach-Object {
    @"
=======================================

Local Address    : $($_.LocalAddress)
Local Port       : $($_.LocalPort)
State            : $($_.State)

Owning Process ID: $($_.OwningProcess)
Process Name     : $($_.ProcessName)
Process Path     : $($_.ProcessPath)

=======================================
"@
} | Out-File -FilePath $txtOutputPath -Width 500

# Export to CSV for structured analysis
$connections | Export-Csv -Path $csvOutputPath -NoTypeInformation

# Confirm file creation
Write-Host "Network connections saved to:"
Write-Host "  - Text file: $txtOutputPath"
Write-Host "  - CSV file: $csvOutputPath"

```

During a live investigation also look at services
* Check for persistance (sample)
```
# Define registry locations to scan
$registryPaths = @(
    "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run",
    "HKLM:\Software\Microsoft\Windows\CurrentVersion\RunOnce",
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run",
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\RunOnce"
)

# Iterate through each registry path and display entries
foreach ($path in $registryPaths) {
    Write-Host "`n🔍 Checking registry path: ${path}" -ForegroundColor Cyan

    if (Test-Path $path) {
        Write-Host "✅ Path exists: ${path}" -ForegroundColor Green

        # Retrieve registry values
        $items = Get-ItemProperty -Path $path -ErrorAction SilentlyContinue

        # Extract properties (excluding system properties)
        $entries = $items.PSObject.Properties | Where-Object { $_.Name -notmatch "^PS" }

        if ($entries) {
            Write-Host "📌 Entries found in `${path}:" -ForegroundColor Yellow
            foreach ($entry in $entries) {
                Write-Host "   🔹 Name  : $($entry.Name)"
                Write-Host "   🔹 Value : $($entry.Value)`n"
            }
        } else {
            Write-Host "⚠ No values found in ${path}" -ForegroundColor DarkYellow
        }
    } else {
        Write-Host "🚫 Path does NOT exist: ${path}" -ForegroundColor Red
    }
}

```
* Get-Services
* Get-CimInstance
