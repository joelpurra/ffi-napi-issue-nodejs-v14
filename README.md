# [ffi-napi-issue-nodejs-v14](https://github.com/joelpurra/ffi-napi-issue-nodejs-v14)

Reproducing an error loading multiple packages which use [`ffi-napi`](https://github.com/node-ffi-napi/ffi-napi) v3.0.1, and perhaps other versions.

Behavior differs between versions of Node.js.

- v10.22.0: no crash.
- v12.18.3: no crash.
- v14.9.0: crash.

See

- https://github.com/node-ffi-napi/node-ffi-napi/issues/96

## Using Node.js v14.9.0

- Requiring `ffi-napi` in one package, then in another package crashes node with a core dump.
  - The crash seems to occur in `ref-napi` in the the second package, even though `ref-napi` is not directly referenced.
  - Requiring `ffi-napi` twice in the same package does not make a difference.
  - Requiring `ref-napi` directly in the two packages does not make a difference.
- Note that both packages are separate directories on disk.
  - This setup produces more than one `node_modules` directory.
  - The issue was discovered when developing interreferenced, not-yet-released packages.
  - Has not been tested with published packages.
- Referencing `package-02` as a package does not make a difference.
  - Using a dependency to `file://../package-02/` in `package-01` crashes.
  - Directly loading `require("../package-02/")` crashes too.
- A workaround is to use `npm link`.
  - Linking to either the `ffi-napi` package or the `ref-napi` package fixes the issue.
  - The file location on disk might matter, if "physically" different files are loaded.

## Steps to verify

```shell
cd ./package-02/
npm install
cd ..

cd ./package-01/
npm install

node index.js
```

<details>

<summary>Sample output</summary>

```text
First package. Step A.
First package. Step B.
Second package. Step A.


#
# Fatal error in , line 0
# Check failed: result.second.
#
#
#
#FailureMessage Object: 0x7ffd9985c4d0
 1: 0xa6f5f1  [node]
 2: 0x19cb8c4 V8_Fatal(char const*, ...) [node]
 3: 0xe58379 v8::internal::GlobalBackingStoreRegistry::Register(std::shared_ptr<v8::internal::BackingStore>) [node]
 4: 0xba38a8 v8::ArrayBuffer::GetBackingStore() [node]
 5: 0x9c1290 napi_get_typedarray_info [node]
 6: 0x7f3c5003a46a  [/some/project/ref-napi-issue-nodejs-v14/package-02/node_modules/ffi-napi/node_modules/ref-napi/prebuilds/linux-x64/node.napi.node]
 7: 0x7f3c5003ad2d  [/some/project/ref-napi-issue-nodejs-v14/package-02/node_modules/ffi-napi/node_modules/ref-napi/prebuilds/linux-x64/node.napi.node]
 8: 0x7f3c50024de7 FFI::WrapPointerImpl(Napi::Env, char*, unsigned long) [/some/project/ref-napi-issue-nodejs-v14/package-02/node_modules/ffi-napi/build/Release/ffi_bindings.node]
 9: 0x7f3c50026b05 FFI::FFI::InitializeBindings(Napi::Env, Napi::Object) [/some/project/ref-napi-issue-nodejs-v14/package-02/node_modules/ffi-napi/build/Release/ffi_bindings.node]
10: 0x7f3c5002781b FFI::InitializeBindings(Napi::CallbackInfo const&) [/some/project/ref-napi-issue-nodejs-v14/package-02/node_modules/ffi-napi/build/Release/ffi_bindings.node]
11: 0x7f3c5002865b Napi::details::CallbackData<Napi::Value (*)(Napi::CallbackInfo const&), Napi::Value>::Wrapper(napi_env__*, napi_callback_info__*) [/some/project/ref-napi-issue-nodejs-v14/package-02/node_modules/ffi-napi/build/Release/ffi_bindings.node]
12: 0x9b8a5f  [node]
13: 0xbe254b  [node]
14: 0xbe3af6  [node]
15: 0xbe4176 v8::internal::Builtin_HandleApiCall(int, unsigned long*, v8::internal::Isolate*) [node]
16: 0x13fe0b9  [node]
[1]    314475 illegal hardware instruction (core dumped)  node index.js
```

</details>

## Workaround

- Run the steps above to verify that the code crashes.
- Download the source code for one of:
  - [`ffi-napi`](https://github.com/node-ffi-napi/node-ffi-napi)
  - [`ref-napi`](https://github.com/node-ffi-napi/ref-napi).

```shell
cd ref-napi # or ffi-napi
npm install
npm link

cd ffi-napi-issue-nodejs-v14

cd package-02
npm link ref-napi # or ffi-napi
cd ..

cd package-01
npm link ref-napi # or ffi-napi

node index.js
```

---

`ffi-napi-issue-nodejs-v14` Copyright &copy; 2020 [Joel Purra](https://joelpurra.com/). Released under [MIT License](https://opensource.org/licenses/MIT). [Your donations are appreciated!](https://joelpurra.com/donate/)
