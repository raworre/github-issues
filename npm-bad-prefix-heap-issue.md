# NPM Uses All Available System Memory When Prefix is Invalid

When the path listed in the `.npmrc` file with the `prefix` key does not exist
any NPM commands will return the following error.

```sh
FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory
```

## Steps to Repeat

1. Place `prefix` entry in `.npmrc` file in `C:\Users\{user}` that points to a
   non-existent drive on the local system (In my case it was `F:`, there is no
   drive mapped to the letter `F` on my machine).
1. Run `npm -v` to check NPM version

Windows Task Manager shows the following memory usage for the
"Node.js: Server-side JavaScript" process.
![Task Manager](task-manager.png)

The following is returned after 2GB of system memory are consumed by the
"Node.js: Server-side JavaScript" Windows process.

```sh
FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory
 1: 00007FF6A06B232F napi_wrap+124543
 2: 00007FF6A06536A6 v8::base::CPU::has_sse+34502
 3: 00007FF6A0654366 v8::base::CPU::has_sse+37766
 4: 00007FF6A0E58C5E v8::Isolate::ReportExternalAllocationLimitReached+94
 5: 00007FF6A0E40CA1 v8::SharedArrayBuffer::Externalize+833
 6: 00007FF6A0D0E56C v8::internal::Heap::EphemeronKeyWriteBarrierFromCode+1436
 7: 00007FF6A0D19910 v8::internal::Heap::ProtectUnprotectedMemoryChunks+1312
 8: 00007FF6A0D16444 v8::internal::Heap::PageFlagsAreConsistent+3204
 9: 00007FF6A0D0BCD3 v8::internal::Heap::CollectGarbage+1283
10: 00007FF6A0D0E6BA v8::internal::Heap::EphemeronKeyWriteBarrierFromCode+1770
11: 00007FF6A0D04D63 v8::internal::IncrementalMarking::WasActivated+467
12: 00007FF6A05FF70C v8::internal::interpreter::BytecodeLabel::bind+119964
13: 00007FF6A05FE6A1 v8::internal::interpreter::BytecodeLabel::bind+115761
14: 00007FF6A06FB61B uv_async_send+331
15: 00007FF6A06FADBC uv_loop_init+1212
16: 00007FF6A06FAF84 uv_run+244
17: 00007FF6A061C8A2 v8::internal::Scope::locals+30978
18: 00007FF6A067A8B3 node::Start+275
19: 00007FF6A053674C RC4_options+339532
20: 00007FF6A1337728 v8::internal::SetupIsolateDelegate::SetupHeap+1301368
21: 00007FFA8ECE7BD4 BaseThreadInitThunk+20
22: 00007FFA90C8CED1 RtlUserThreadStart+33
6.13.4
```

## Expected Behavior

In a case where a bad prefix is provided in the `.npmrc` file, NPM should
fail fast letting the user know that the expected `prefix` path is invalid,
rather than consuming available system memory until the internal Node heap
limit is hit.

## System Information

* Windows 10&mdash;10.0.18363 Build 18363
* NodeJS&mdash;12.14.1
* NPM&mdash;6.13.4
