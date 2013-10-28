protostore - A very simple CRUD storage system for protobuf messages
====================================================================

Google Protobuf Serialisation Data Access Object Library.

Supports serialisation of protobuf messages to sql columnar storage with
Create Read Update Delete semantics, primary key on UUID or auto-increment and 
secondary indexes.

The goal of this project is to provide a very light weight mechanism to support
CRUD operations of any proto message, primarily to avoid writing SQL.

In java the library has already helped many times with database and object
refactorings. For example to rename a column in your database:

1. Generate your proto java.
2. Using your IDE rename get<Field> and has<Field> in the message, set<Field>
    and clear<Field> resulting in all code migrating to the new format.
3. Modify you proto field name.
4. Re-generate your proto.
5. Refactor your database.
6. Deploy your updated proto access.
7. Clean up your database rubbish columns.
8. Joy. No additional tests. No additional SQL.

Many database and object refactorings follow a similar pattern.

Features
--------
* Store protobuf to SQL.
* Write proto messages for legacy database tables without having to write sql.
* Primary index support with auto increment and java UUID key support.
* Secondary indexes on additional field values.
* Ordered data [Urn field store only for now].

Limitations
-----------
* MySQL like syntax for now.
* Limited query ability.
* Only supports protobuf 2.4.1 style syntax.
* Only supports iteration of values, no aggregation functions.
* No roll back on build error.


Usage
-----

Assuming that you are simply using the proto storage to a legacy SQL database
you have two options:

* DbFieldCrudStore: Used for auto incremented keyed tables.
* DbUrnFieldStore: Uses Java UUID for key value which is better for large web apps
    due to avoidance of lock / centralised auto-increment.

Both store examples follow a similar usage pattern:

```java

  CrudStore<MyMessage> store = new DbUrnFieldStore.Builder<MyMessage>()
      .setConnection(sqlConnection)
      .setPrototype(MyMessage.newBuilder())
      .setTableName("MyMessageTable")
      .setUrnColumn("uuid")
      .addIndexField("secondaryIndex")
      .setSortOrder("someSortField")
      .build();

```

Within your framework you can use any construction / injection needed that
calls the relevant type of storage driver for your database. If you are using
regular INTEGER auto ID columns use the DbFieldCrudStore class. Note that
different implementations will provide different feature sets based on the
builder functions but the two main both support primary and secondary indexes.

The read method on a crud store is the only 'advanced' interface, all others are
pretty much as they seem. For read, which support basic querying you can set:

* The builder ID/URN value and you will read the record for that ID.
* The value of a secondary key and all matches will be read.
* No values in the builder parameter and you will get back 'ALL' results.

This corresponds to

```java

  // read by key field
  CrudIterator<MyMessage> keyIterator = store.read(
      MyMessage.newBuilder()
      .setUrn("some-key"));

  // read secondary key
  CrudIterator<MyMessage> secondaryIterator = store.read(
      MyMessage.newBuilder()
      .setSecondary("secondary-key"));

  // read all
  CrudIterator<MyMessage> all = store.read(MyMessage.newBuilder());

```

Note that create in each driver type will set the ID or urn field for you based
on satisfying database constraints.

Note that you must set the ID / URN on update.



