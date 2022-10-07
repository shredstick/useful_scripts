
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