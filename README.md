# Info

```
.
├── app
│   ├── main.ts
├── proto1
│   └── proto1.proto
└── proto2
    └── proto2.proto
```

`proto1` exposes a standalone protobuf `js_library` `//proto1:proto1`.

`proto2` exposes a protobuf `js_library` `//proto2:proto2` but depends on `proto1` at `protoc` compile time (`import "proto1/proto1.protoc"`).

`bazel build //proto2` succedes, but it can't be used as a dependency to `//app:main` because the paths generated for the `require` are relative, and can't be resolved when the `ts_project` is build:

```
$ bazel build //app:main
INFO: Analyzed target //app:main (0 packages loaded, 0 targets configured).
INFO: Found 1 target...
ERROR: /home/joe/code/proto_test/app/BUILD.bazel:4:11: Compiling TypeScript project //app:main [tsc -p tsconfig.json] failed: (Exit 2): tsc.sh failed: error executing command bazel-out/host/bin/external/npm/typescript/bin/tsc.sh --project tsconfig.json --outDir bazel-out/k8-fastbuild/bin/app --rootDir app ... (remaining 1 argument(s) skipped)

Use --sandbox_debug to see verbose messages from the sandbox
bazel-out/k8-fastbuild/bin/proto2/proto2/proto2_pb.d.ts(5,35): error TS2307: Cannot find module '../proto1/proto1_pb' or its corresponding type declarations.
Target //app:main failed to build
Use --verbose_failures to see the command lines of failed build steps.
INFO: Elapsed time: 2.239s, Critical Path: 2.03s
INFO: 2 processes: 2 internal.
FAILED: Build did NOT complete successfully
```

The generated code in that causes this error in `proto2_pb.js` is `var proto1_proto1_pb = require('../proto1/proto1_pb.js');`