![Green White And Blue Earth Day Textured Illustration Banner (Landscape)](https://user-images.githubusercontent.com/60538942/160297488-4efd6cb9-fc99-4db5-8699-de1e3f2c93ed.png)



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

To shut down the Mongodb `mongo admin --port 27001 --eval "db.shutdownServer()"` or navigate to `admin` database and run `db.shutdownServer()`

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
2. Run the below command
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
1. `mongo` or `mongo --port <mongoDB port>` or along with any other flags as required
then
2. `use admin`
3. `db.auth("globalAdminUser", "<password_that_was_set>")`
it should return 1
#### or 
`mongo admin --port 27000 --username 'globalAdminUser' --password '<password_that_was_set>'`

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
when clicked enter, it should show success response along with given username, role.

#### check the available users: 
`db.getUsers()`

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

**Built-In Role: userAdmin** <br>
This role grants the following privileges:
- changeCustomData
- changePassword
- createRole
- createUser
- dropRole
- dropUser
- grantRole
- revokeRole
- setAuthenticationRestriction
- viewRole
- viewUser

## User-Defined Roles
We can create custom roles and give access to selected privileges as below.
```
db.createRole({
  role: "grantRevokeViewRolesAnyDatabase",
  privileges: [
    {
      resource: {db: "", collection: ""},
      actions: ["grantRole, revokeRole, viewRole"],
    },
  ],
  roles: [],
})
```
- here if db: "<database_name>" and collection: "", it gives the privileges on all collections inside the specified database
- roles: [] is given here, but we can give built-in roles as well. This will create a hybrid role with the privileges of built-in roles privileges and the privileges we provided
- Another Example to create a role with `find` and `insert` privileges over the `transactions` database
```
db.createRole({
  role: "insertAndFindTransactions",
  privileges: [
    {
      resource:{db: "transactions", collection: ""},
      actions: ["find", "insert"]
     },
  ],
  roles: [],
})
```

#### Check the available roles
`db.getRoles()`

**Note:** The roles can be created only by the user with createRole access, ensure that before running the above command to avoid errors
## Multiple Methods for updating the user Information
The `updateUser` method can be used to modify any user field in the user document
- It is not limited to any role
- The user who is running hte query still requires all the desired privileges (typically run by user administrators)
```
db.updateUser(
  "<userName>".
  {
    roles: [
      { role: "<role>", db: "<db>"},
    ],
    pwd: "<password>",
    mechanisms: ["<auth-mechanism>"],
    ............
  }
)
```
**Note:** The roles specified here will override all the role values assigned earlier.

Ex: To update the password of the user
`db.updateUser("readWriteInventoryUser", {pwd: "6c%dbe&7dc!ee1#d"})`

# Internal Authentication
Internal Authentication verifies the identity of one MongoDB instance to another.

- When Internal Authentication is not enabled, theoretically a hacker can add a `rogue member` to the replica set and ciphon any data being replicated from the primary.

### How could an attacker add a rogue member?
1. Connect to the primary node in the replica set
2. Authenticate to the primary as a `clusterAdmin`, or some user with privileges to add a member to the replica set

## Internal Authentication with Key files
This can be achieved by enforcing SCRAM(Salted Challenge Response Authentication Mechanism) which accepts key files instead of a password. A key files is a file with a very long password.
This key file can be copied on to each member of the replica set and must be presented to other members during authentication. 

**Let's add Authentication at the instance level**
1. Assume the key files are located at `var/mongodb/pki/node1/keyfile`.
2. Now let's create a configuration file to include links to the key files
3. open the mongod_1.conf file and edit the content as below
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
  keyFile: var/mongodb/pki/<node1/node2/node3..>/keyfile
```
**Note:** Adding a keyfile implicitly enforce Authentication for both users and instances in the replica set. So at this step `authorization: enabled` flag is not necessary.

4. Do this for all other instances on the replice set.
5. Now start each member of replica set with new configuration by `mongod -f mongod_1.conf` and `mongod -f mongod_2.conf` and so on.
6. Then connect to one of the members of the repliceset and initiate by `rs.initiate()`
7. Now you can create the very first user according to Localhost Exception
8. And authorize using the credentials

*Command to initiate:*

```
rs.initiate({
  _id: "<any string>",
  version: 1,
  members: [
    { _id: 0, host: "localhost:27000" }, // host: host Ip address if the replica set is distributed over different Ip address
    { _id: 1, host: "localhost:27001" },
    { _id: 2, host: "localhost:27002" },
    ....
  ],
})
```
## x509: An Alternative to SCRAM
This requires certificates as the credential. These are the files that are primarily used to enable Transport Layer Security(TLS) between two servers.
TLS encrypts data sent over the network, so outsiders cannot read it.

### x509 Internal Authentication
- Each server will have an x509 certificate which will be used to authenticate to the other member
- This is also used for encryption between each member of replica set

