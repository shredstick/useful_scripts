
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