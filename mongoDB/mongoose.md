# performance improve

* When your application starts up, Mongoose automatically calls `ensureIndex` for each defined index in your schema. Mongoose will call `ensureIndex` for each index sequentially, and emit an 'index' event on the model when all the `ensureIndex` calls succeeded or when there was an error. While nice for development, it is recommended this behavior be disabled in production since index creation can cause a [significant performance impact](http://docs.mongodb.org/manual/core/indexes/#index-creation-operations). Disable the behavior by setting the `autoIndex` option of your schema to `false`, or globally on the connection by setting the option `config.autoIndex` to `false`.

  ```Js
  mongoose.connect('mongodb://user:pass@localhost:port/database', { config: { autoIndex: false } });
  // or  
  mongoose.createConnection('mongodb://user:pass@localhost:port/database', { config: { autoIndex: false } });
  // or
  animalSchema.set('autoIndex', false);
  // or
  new Schema({..}, { autoIndex: false });
  ```

  ​



