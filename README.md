# example-replica-set

This is a little example of how to make a replica set using docker-compose.

## Steps

* First generate a [keyfile](https://docs.mongodb.com/manual/tutorial/enforce-keyfile-access-control-in-existing-replica-set/) to be used in the creation of the replicaSet.

```shell script
openssl rand -base64 756 > <path-to-keyfile>
chmod 400 <path-to-keyfile>
```

* Run the docker-compose with:

```shell script
docker-compose up -d
```

* Connect to the machines using:

```shell script
mongo mongodb://admin:admin@<ip>:27010/admin
```

Also you can connect using docker:

```shell script
docker exec -it mongo-1 bash

mongo-1@machine:~$ mongo
```

* In mongo-shell we need to create the replicaSet

```
rs.initiate(
    {
      _id : 'rs',
      members: [
        { _id : 0, host : "<ip-machine>:27010" },
        { _id : 1, host : "<ip-machine>:27011" }
      ]
    }
);
```
After this command the response should be OK and you'll have a rs ready.

### Useful commands

* Command for create a user to handle a db:

In mongo-shell

```
use dbExample

db.createUser({
    user: "userExample",
    pwd: "<pass>",
    roles: [
        {
            role: "readWrite",
            db: "dbExample"
        }
    ],
    mechanisms:[
    "SCRAM-SHA-1"
    ]
});
```

* If we need a opLog user we can create one in admin db:

We need to log with admin for this purpose.

```
db.createUser({
    user: "oplog",
    pwd: "<pass>",
    roles: [
        {
            role: "read",
            db: "local"
        }
    ],
    mechanisms:[
    "SCRAM-SHA-1"
    ]
});
```

* For meteor users they can set this env variables to set his mongo URL's

```shell script
export MONGO_URL='mongodb://userExample:<pass>@<ip-machine-with-rs>:27010/dbExample'
export MONGO_OPLOG_URL='mongodb://oplog:<pass>@<ip-machine-with-rs>:27010,<ip-machine-with-rs>:27011/local?authSource=admin&replicaSet=rs'
```

