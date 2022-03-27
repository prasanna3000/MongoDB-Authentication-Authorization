# Setup Authentication
1. Create or Edit mongod.conf
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

## Localhost Exception
- By default, there are no users authorized. If in this scenario the authorization flag is set, one can't create users without authenticating themselves first. 
To handle this deadlock situation there is localhost rule that will allow to create One(1) new user.
- If there is a user already exists in the database, localhost exception cannot be used
- Must be connected via localhost (it means access to the physical machine, usually an administrator)

### Note: The first user must always be a global user in the admin database

## Let's create an Admin user in the database `admin`
1. `use admin`
2.
 ```db.createUser(
 {
   user: "globalAdminUser", // could be any name
   pwd: passwordPrompt(), // prompts you to enter password, you can enter the plain password text as well here
   roles: [
     { role: "userAdminAnyDatabase", db: "admin"},
   ]
 })
 ```
 3. On click enter it should show Successfully added user : followed by the user name and the role details
 4. If we try to create a new user, it throws error at this instant

## Authenticate the admin user
`db.auth("globalAdminUser", "<password_that_was_set>")`
it should return 1

## To create a user with cluster related permissions
This is very powerful and is available only with admin database
```
db.createUser({
  user: "clusterAdminAny",
  pwd: "some_password",
  roles: [
    {role: "clusterAdmin", db: "admin"},
  ]
})
```
when clicked enter, it should show success response along with given username, role

#### Required Access
1. To create a new user in a database, you must have the `createUser` action on that database resource.
2. To grant roles to a user, you must have the `grantRole` action on the role's database.
The `userAdmin` and `userAdminAnyDatabase` built-in roles provide `createUser` and `grantRole` actions on their respective resources.

To know more visit: [MongoDB DOfficial Documentation](https://www.mongodb.com/docs/manual/reference/method/db.createUser/)

# Role-Based Access
## Built-in Roles
1. userAdminAnyDatabase (available only in admin database
2. userAdmin (available in any database)
3. dbAdmin (availabel in any database)

There are two ways to assign roles to the user
1. using db.createUser({}) and specifying the roles here itself
2. using db.grantRolesToUser({}) but in this case the `roles:[]` must have had been empty array when this user was created
```
db.grantRolesToUser(
 "<userName>",
 [
  {role: "userAdmin", db: "inventory" },
 ])
```
Note: MongoDB provide 15 built in roles for role-based access control
- User can use them or define new privileges to grant minimum access also called `Principle of Least Privilege`
### Least Privilege Principle
`The principle of Least Privege says that the users should have the least privilege required for their intended purpose`

