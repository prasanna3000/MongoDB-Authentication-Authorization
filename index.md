# Setup Authentication
1. Edit mongod.service
```
storage:
  dbPath: data/db(path to the database)
net:
  bindIp: localhost(ip addrees where mongdb is running)
  port: 27001
systemLog:
  destination: file
  path: logs/mongod.log(path to log file)
  logAppend: true
processManagement:
  fork: true
security:
  authorization: enabled(enables both authorization and authentication)
```

After creating/editing this file. Shut down the MongoDb and start again. When it starts, it asks for Authentication.
So be sure to create users before enabling the security flag.
To shut down the Mongodb `mongo admin --port 27001 --eval "db.shutdownServer()"`

2. Update the Configuration
`mongod -f mongod.conf`

3. Start the server again `mongod --port 27001` and try to run any sample command and it throws error

