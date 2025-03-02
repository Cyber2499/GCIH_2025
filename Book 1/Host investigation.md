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

During a live investigation also look at services
* Get-Services
* Get-CimInstance
