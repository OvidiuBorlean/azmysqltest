# Azure MySQL Test

When customer complains about connectivity issues towards the Azure MySQL endpoints, we developed a simple Python script that is continuously testing respective endpoint and save the timestamped output both in stdout and local file.

# Prerequisites

For running this script, we need to deploy a testing Pod on AKS cluster where we will upload our Python script along with the necessary libraries. For our testing, we used the classic nginx image.

```
kubectl run nginx --image=nginx
kubectl exec -it nginx -- bash
```

# Installation

After our Nginx pod is in running state and we connect to shell, we need to initialize repositories:
**apt update**

We will install the following packages:

Python3
Python3-pip
mysql-connector-python (PIP)

```
apt install python3 -y apt install python3-pip -y pip install mysql-connector 
```

Creating our folder where our script and logs will run:

```
mkdir /app
cd /app

cat << EOF ./mysqltest.py
import mysql.connector
from mysql.connector import errorcode
import os
import time
from datetime import datetime

def checkMySql():
  try:
    now = datetime.now()
    dt_string = now.strftime("%d/%m/%Y %H:%M:%S")
    conn = mysql.connector.connect(**config)
    print(dt_string + " " + "Connection established to MySQL Server")
    conn.close()
    file1= open("mysqltest.log","a")
    content = dt_string + " " + "Connection established to MySQL Server"
    file1.write(content)
    file1.write("\n")
    file1.close()
  except mysql.connector.Error as err:
    if err.errno == errorcode.ER_ACCESS_DENIED_ERROR:
     print("Something is wrong with the user name or password")
     file1= open("mysqltest.log","a")
     content = dt_string + " " + "Something is wrong with the user name or password"
     file1.write(content)
     file1.write("\n")
     file1.close()
    elif err.errno == errorcode.ER_BAD_DB_ERROR:
      print("Database does not exist")
      file1= open("mysqltest.log","a")
      content = dt_string + " " + "Database does not exist"
      file1.write(content)
      file1.write("\n")
      file1.close()
    else:
      print(err)
      file1= open("mysqltest.log","a")
      content = dt_string + " " + str(err)
      file1.write(content)
      file1.write("\n")
      file1.close()
    #else:
    #  cursor = conn.cursor()
    #  cursor.execute("DROP TABLE IF EXISTS inventory;")
    #  print("Finished dropping table (if existed).")
    #  cursor.execute("CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);")
    #  print("Finished creating table.")
    #  # Insert some data into table
    #  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("banana", 150))
    #  print("Inserted",cursor.rowcount,"row(s) of data.")
    #  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("orange", 154))
    #  print("Inserted",cursor.rowcount,"row(s) of data.")
    #  cursor.execute("INSERT INTO inventory (name, quantity) VALUES (%s, %s);", ("apple", 100))
    #  print("Inserted",cursor.rowcount,"row(s) of data.")
    # Cleanup
    # conn.commit()
    #  cursor.close()
    # conn.close()
    # print("Done.")

if __name__ == '__main__':
  myHostname = os.getenv('SQLSERVER')
  mySQLUser = os.getenv("SQLUSER")
  myPassword = os.environ.get('SQLPASS')
  myDB = os.environ.get('SQLDB')
  interval = os.environ.get('TIMEINTERVAL')
  config = {
  'host':myHostname,
  'user':mySQLUser,
  'password':myPassword,
  'database':myDB,
  'client_flags': [mysql.connector.ClientFlag.SSL],
  'ssl_ca': '/home/user/mysql/DigiCertGlobalRootG2.crt.pem'
  }
  print("Starting MySQL Connectivity Check:...")
  while True:
    checkMySql()
    time.sleep(int(interval))
EOF
```

Before running our script, we need to add MySQL connectivity details into environment variables. We can do this as follows:

```
export SQLSERVER="yourMySQLSERVER"
export SQLUSER="yourMySQLUser"
export SQLPASS="yourMySQLPassword"
export DBNAME="yourDBName"
export TIMEINETERVAL="5" # Desired time interval in sec between tests
```

