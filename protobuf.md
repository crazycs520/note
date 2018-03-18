```shell
protoc -I=/Users/cs/code/go/src/protobuf --js_out=/Users/cs/code/go/src/protobuf /Users/cs/code/go/src/protobuf/addressbook.proto


protoc --js_out=library=myprotos_lib.js,binary:. addressbook.proto
protoc --js_out=import_style=commonjs,binary:. addressbook.proto

protoc --go_out=. addressbook.proto
```

