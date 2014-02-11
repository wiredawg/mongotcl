This is MongoTcl, a Tcl C extension providing an interface to the MongoDB C driver.
===

MongoTcl provides a Tcl interface to the MongoDB C API.

FA Note
---
If pursued far enough, when this is solid enough it will be open sourced.

Building
---

    autoconf
    configure
    make
    sudo make install

For FreeBSD, something like

    ./configure --with-tcl=/usr/local/lib/tcl8.5  --mandir=/usr/local/man --enable-symbols
 
Accessing from Tcl
---

    package require mongo


MongoTcl objects
---

MongoTcl provides three object creation commands...

* ::mongo::mongo, to access MongoDB databases, query and update them
* ::mongo::bson, to create and manipulate bson objects
* $mongo cursor, to create a cursor object from a MongoDB object

BSON object
---

status: constructing bson is solid.  deconstructing it is untested to nonexistent.

BSON stands for Binary JSON, is a binary-encoded serialization of JSON-like documents.  It has a JSON-like structure but is extended to support data types beyond the JSON spec, like it has a binary data type.

It's intended to be lightweight, traversable and efficient and it's the primary data representation for MongoDB.

More about bson at http://bsonspec.org/

MongoDB use of BSON at http://docs.mongodb.org/meta-driver/latest/legacy/bson/

MongoTcl has a bson object and the bson creator is invoked to create bson objects, siilarly to iTcl objects

```tcl
    ::mongo::bson create
```

or

```tcl
    set obj [::mongo::bson create #auto]
```

Methods of the BSON object
---

* $bson init

Initialize or reinitialize the bson object.  It's initialized upon creation.

* $bson string $key $value

Append a key and value to the bson object.

* $bson int $key $value

Append a key and value to the bson object where the value is a number.

* $bson double $key $value

Append a key and value to the bson object where the value is a double-precision floating point.

* $bson bool $key $value

Append a key and a boolean value to the bson object.

* $bson clock $key $epoch

Append a key and epoch to the bson object.  Stored in milliseconds but program multiplies by 100.  Probably shouldn't and you should use clock clicks -milliseconds for current time.

* $bson null $key

Append a key and a null.

* $bson undefined $key

Append a key and an undefined.

* $bson kvlist $list

Import a list of key-value pairs.  Values are encoded as strings.  

* $bson binary key type $binaryData

Append a key and binary data.  Type can be ''generic'', ''function'', ''uuid'', ''md5'', ''user_defined''.

Typical usage

```tcl
    $bson kvlist [array get arrayName]
```

List must be contain an even number of elements.

* $bson bson key bsonObject

Append a key and contents of a bson object to the bson object.

* $bson start_array

Begin an array.

Note that the docs say that it's still key value but the keys need to be 0, 1, 2, etc, so for now you have to roll your own although this could easily be coded as a proc or C method.

* $bson finish_array

* $bson new_oid $field

* $bson start_object

Start a subobject.

* $bson end_object

End a subobject.

* $bson finish

Finish the bson object.  I guess this rounds it out and completes it.

* $bson new_oid $key

Append a key and a bson-library-generated oid to the bson object.

* $bson to_list

Enumerate bson object as a list.

* $bson print

Print is for debugging only, it sort of shows you what's in the bson object.

All of the methods can be combined in a single command, for example:

```tcl
$bson init string "name" "Joe" int "age" 33 finish
```


Methods of mongo, the MongoDB interface object
---

```tcl
    set mongo [::mongo::mongo create #auto]
```

* $mongo init

Initialize or reinitialize the mongo object.  Like bson, it's initialize upon creation.

* $mongo insert $namespace $bson

Insert the specified bson object in the database with the specified namespace.

* $mongo update $namespace $condBson $opBson ?updateType?

Update the specified bson object.  condBson is the update query in bson.  opBson is the bson update data.  The update type can be ''basic'', ''multi'', ''upsert''.  ''basic'' is used if update type isn't specified.

* $mongo insert_batch $namespace $bsonObjectList

There's a compile warning on this.  It's probably coredump.

* $mongo remove $namespace $bson

Removes a document from a MongoDB server.  bson is the bson query.

* $mongo cursor

* $mongo find $namespace $bsonQuery $bsonFields $limit $skip $options

* $mongo count $db $collection

Return a count of object in the collection.

* $mongo last_error $db

Return the last error.

* $mongo prev_error $db

Return the previous error.

* $mongo write_concern concern_option ?concern_option?

The write_concern method takes one or more options.  ''ignore_errors'' says to ignore errors.  ''unacknowledged'', the default, says to write unacknowledged, which ''acknowledged'' says to write acknowledged.

In addition to the above options, which are one-of-three, ''replica_acknowledged'' adds that I guess a quorm of replicas have acknowledged the write, and ''journaled'' requires it to have been written to disk before the call returns.

* $mongo create_index $namespace $keyBson $outBson ?optionList?

Create an index.  This can easily done from some CLI that comes with MongoDB, anyway.

* $mongo set_op_timeout $ms

Set operation timeout in milliseconds.

* $mongo client $address $port

Define a connection to an address and port.  A later C API (yet to be seen on FreeBSD ports and not configuring cleanly natively) supports a URL-type structure.

* $mongo reconnect

Reconnect to the database.

* $mongo disconnect

Disconnect from the database.

* $mongo check_connection

Check the database connection status.  Returns 0 or 1.

* $mongo replica_set_init

* $mongo replica_set_add_seed $address $port

* $mongo replica_set_client

* $mongo clear_errors

Clear errors.

* $mongo authenticate $db $user $pass

Authenticate to the named database.  MongoDB can be run without authentication.

* $mongo add_user $db $user $pass

Add a user and specify their password.  Again the CLI may be better for this.

* $mongo drop_collection $db $collect

Drop a collection.

* $mongo drop_db $db

Drop a database.

Example
---

```tcl
    $bson init new_oid _id string "name" "Joe" int "age" 33 finish

    $mongo insert "tutorial.persons" $bson
```

When you're done using the bson object, destroy it by doing a

    rename $bson ""

When you're done using the mongodb object, destroy it similarly.
