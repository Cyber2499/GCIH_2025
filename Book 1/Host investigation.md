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

# Gather process information
$processes = Get-CimInstance Win32_Process | ForEach-Object {
    $processName = (Get-Process -Id $_.ProcessId -ErrorAction SilentlyContinue).ProcessName
    $parentProcessName = (Get-Process -Id $_.ParentProcessId -ErrorAction SilentlyContinue).ProcessName

    [PSCustomObject]@{
        ParentProcessId   = $_.ParentProcessId
        ParentProcessName = $parentProcessName
        ProcessId         = $_.ProcessId
        ProcessName       = $processName
        CommandLine       = $_.CommandLine
    }
}

# Export to a neatly formatted text file (list format)
$processes | ForEach-Object {
    @"
=======================================
Process ID      : $($_.ProcessId)
Process Name    : $($_.ProcessName)
Parent ID       : $($_.ParentProcessId)
Parent Name     : $($_.ParentProcessName)
Command Line    : $($_.CommandLine)
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
