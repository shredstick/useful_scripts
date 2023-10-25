
# Scripts

This is my small collection of useful Scripts.

  

## Python connect to SQL

  

In this example we connect to a SQL Server and submit a query.

  

```bash

import pyodbc

# Some other example server values are

# server = 'localhost\sqlexpress' # for a named instance

# server = 'myserver,port' # to specify an alternate port

server = 'tcp:myserver.database.windows.net'

database = 'mydb'

username = 'myusername'

password = 'mypassword'

# ENCRYPT defaults to yes starting in ODBC Driver 18. It's good to always specify ENCRYPT=yes on the client side to avoid MITM attacks.

cnxn = pyodbc.connect('DRIVER={ODBC Driver 18 for SQL Server};SERVER='+server+';DATABASE='+database+';ENCRYPT=yes;UID='+username+';PWD='+ password)

cursor = cnxn.cursor()

  

#Sample select query

cursor.execute("SELECT @@version;")

row = cursor.fetchone()

while  row:

print(row[0])

row = cursor.fetchone()

  
  

#Sample insert query

count = cursor.execute("""

INSERT INTO SalesLT.Product (Name, ProductNumber, StandardCost, ListPrice, SellStartDate)

VALUES (?,?,?,?,?)""",

'SQL Server Express New 20', 'SQLEXPRESS New 20', 0, 0, CURRENT_TIMESTAMP).rowcount

cnxn.commit()

print('Rows inserted: ' + str(count))

```


## PowerShell check CPU Temperature

```bash
function GetCPUTemperatureInCelsius {
	$Temp = 99999.9 # unsupported
	if ($IsLinux) {
		if (Test-Path "/sys/class/thermal/thermal_zone0/temp" -pathType leaf) {
			[int]$IntTemp = Get-Content "/sys/class/thermal/thermal_zone0/temp"
			$Temp = [math]::round($IntTemp / 1000.0, 1)
		}
	} else {
		$Objects = Get-WmiObject -Query "SELECT * FROM Win32_PerfFormattedData_Counters_ThermalZoneInformation" -Namespace "root/CIMV2"
		foreach ($Obj in $Objects) {
			$HiPrec = $Obj.HighPrecisionTemperature
			$Temp = [math]::round($HiPrec / 100.0, 1)
		}
	}
	return $Temp;
}

try {
	$Temp = GetCPUTemperatureInCelsius
	if ($Temp -eq 99999.9) {
		"⚠️ CPU temperature query is unsupported."
	} elseif ($Temp -gt 80) {
		"⚠️ CPU is too hot at $($Temp)°C!"
	} elseif ($Temp -gt 50) {
		"✅ CPU is $($Temp)°C hot."
	} elseif ($Temp -gt 0) {
		"✅ CPU is $($Temp)°C warm."
	} elseif ($Temp -gt -20) {
		"✅ CPU is $($Temp)°C cold."
	} else {
		"⚠️ CPU is too cold at $($Temp)°C!"
	}
	exit 0 # success
} catch {
	"⚠️ Error in line $($_.InvocationInfo.ScriptLineNumber): $($Error[0])"
	exit 1
}

```


## PowerShell download a File

```bash
param([string]$URL = "")

try {
	if ($URL -eq "") { $URL = read-host "Enter file URL to download" }

	$StopWatch = [system.diagnostics.stopwatch]::startNew()

	& wget --version
	if ($lastExitCode -ne "0") { throw "Can't execute 'wget' - make sure wget is installed and available" }

	& wget --mirror --convert-links --adjust-extension --page-requisites --no-parent $URL --directory-prefix . --no-verbose
	if ($lastExitCode -ne "0") { throw "Can't execute 'wget --mirror $URL'" }

	[int]$Elapsed = $StopWatch.Elapsed.TotalSeconds
	"✔️ downloaded file from $URL in $Elapsed sec"
	exit 0 # success
} catch {
	"⚠️ Error in line $($_.InvocationInfo.ScriptLineNumber): $($Error[0])"
	exit 1
}


```

## PowerShell list installed Applications

```bash
try {
	if ($IsLinux) {
		& snap list
	} else {
		Get-AppxPackage | Select-Object Name,Version | Format-Table -autoSize
	}
	exit 0 # success
} catch {
	"⚠️ Error in line $($_.InvocationInfo.ScriptLineNumber): $($Error[0])"
	exit 1
}
```

## PowerShell monitor a server

```bash
Function Monitor-ServerPerformance {
    param (
        [string]$ServerName
    )
    
    $cpuUsage = Get-WmiObject Win32_Processor -ComputerName $ServerName | Measure-Object -Property LoadPercentage -Average | Select Average
    $memoryUsage = Get-WmiObject Win32_OperatingSystem -ComputerName $ServerName | Select-Object @{Name = "MemoryUsage"; Expression = {("{0:N2}" -f ((($_.TotalVisibleMemorySize - $_.FreePhysicalMemory) / $_.TotalVisibleMemorySize) * 100))}}
    $diskUsage = Get-WmiObject Win32_LogicalDisk -Filter "DriveType=3" -ComputerName $ServerName | Select-Object DeviceID, @{Name = "FreeSpace(GB)"; Expression = {[math]::Round($_.FreeSpace / 1GB, 2)}}, @{Name = "Size(GB)"; Expression = {[math]::Round($_.Size / 1GB, 2)}}

    Write-Host "Server Performance Metrics for $ServerName :"
    Write-Host "CPU Usage: $($cpuUsage.Average) %"
    Write-Host "Memory Usage: $($memoryUsage.MemoryUsage) %"
    Write-Host "Disk Usage:"
    $diskUsage | ForEach-Object {
        Write-Host "Drive $($_.DeviceID): Free Space $($($_.'FreeSpace(GB)')) GB, Total Size $($($_.'Size(GB)')) GB"
    }
}

```

## Check Firewall & Defender

```bash
if (-NOT ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) {
    Write-Warning "You need to run this script as an Administrator"
    Exit
}

$firewallStatus = Get-NetFirewallProfile | Select Name, Enabled
if ($firewallStatus.Enabled -eq 'True') {
    Write-Host "Windows Firewall is enabled."
} else {
    Write-Host "Windows Firewall is not enabled. Please enable it for improved security."
}

# Example: Checking if antivirus is installed and up to date
$antivirusStatus = Get-WmiObject -Namespace "root/SecurityCenter2" -Class AntiVirusProduct | Select DisplayName, ProductState
if ($antivirusStatus -ne $null) {
    Write-Host "Antivirus software found: $($antivirusStatus.DisplayName)"
} else {
    Write-Host "No antivirus software found. Install an antivirus program for better security."
}

```