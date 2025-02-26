During a live investigation look at processes via:
* Get-Process
  * Enrich with Get-CimInstance (get parent process and command line)

During a live investigation also look at current connections:
* Get-NetTCPConnection
  * Enrich with Get-Process

During a live investigation also look at services
* Get-Services
* Get-CimInstance
