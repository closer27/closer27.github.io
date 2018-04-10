---
layout: post
title: "앵귤러 프로젝트 빌드시  “CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory” 에러"
date: 2018-04-09
categories: frontend
tags:
- angular
- node
- build
comment: yes
---

그동안 개발 된 컴포넌트들을 일괄적으로 lazy loading을 적용하고 배포하려니 발생하는 갑자기 문제인데, 아래와 같은 에러 메세지와 함께 빌드가 되지 않았다. 처음엔 어딘가 모듈 로딩이 잘못 걸려서 무한 루프가 돌면서 빌드가 되지 않는게 아닐까 싶었는데

```bash
> ng build --prod --aot --env=prod-pi

 92% chunk asset optimization
<--- Last few GCs --->

[23640:0x104001600]   120292 ms: Mark-sweep 1218.8 (1443.7) -> 1218.8 (1444.2) MB, 801.8 / 0.0 ms  allocation failure GC in old space requested
[23640:0x104001600]   121119 ms: Mark-sweep 1218.8 (1444.2) -> 1218.8 (1389.2) MB, 827.6 / 0.0 ms  last resort GC in old space requested
[23640:0x104001600]   121933 ms: Mark-sweep 1218.8 (1389.2) -> 1218.8 (1372.7) MB, 814.0 / 0.0 ms  last resort GC in old space requested


<--- JS stacktrace --->

==== JS stack trace =========================================

Security context: 0x271a8da54d9 <JSObject>
    1: _send [internal/child_process.js:694] [bytecode=0x2719e045ff9 offset=622](this=0x2710a0e4bd9 <ChildProcess map = 0x271e23bae89>,message=0x2713c555091 <Object map = 0x271e23bac21>,handle=0x271b27822d1 <undefined>,options=0x2713c554c61 <Object map = 0x271e23baf39>,callback=0x271b27822d1 <undefined>)
    2: send [internal/child_process.js:604] [bytecode=0x2719e045741 offset=150](this=0x2710a0...

FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
 1: node::Abort() [/usr/local/bin/node]
 2: node::FatalTryCatch::~FatalTryCatch() [/usr/local/bin/node]
 3: v8::Utils::ReportOOMFailure(char const*, bool) [/usr/local/bin/node]
 4: v8::internal::V8::FatalProcessOutOfMemory(char const*, bool) [/usr/local/bin/node]
 5: v8::internal::Factory::NewRawTwoByteString(int, v8::internal::PretenureFlag) [/usr/local/bin/node]
 6: v8::internal::String::SlowFlatten(v8::internal::Handle<v8::internal::ConsString>, v8::internal::PretenureFlag) [/usr/local/bin/node]
 7: v8::String::WriteUtf8(char*, int, int*, int) const [/usr/local/bin/node]
 8: node::StringBytes::Write(v8::Isolate*, char*, unsigned long, v8::Local<v8::Value>, node::encoding, int*) [/usr/local/bin/node]
 9: int node::StreamBase::WriteString<(node::encoding)1>(v8::FunctionCallbackInfo<v8::Value> const&) [/usr/local/bin/node]
10: void node::StreamBase::JSMethod<node::LibuvStreamWrap, &(int node::StreamBase::WriteString<(node::encoding)1>(v8::FunctionCallbackInfo<v8::Value> const&))>(v8::FunctionCallbackInfo<v8::Value> const&) [/usr/local/bin/node]
11: v8::internal::FunctionCallbackArguments::Call(void (*)(v8::FunctionCallbackInfo<v8::Value> const&)) [/usr/local/bin/node]
12: v8::internal::MaybeHandle<v8::internal::Object> v8::internal::(anonymous namespace)::HandleApiCallHelper<false>(v8::internal::Isolate*, v8::internal::Handle<v8::internal::HeapObject>, v8::internal::Handle<v8::internal::HeapObject>, v8::internal::Handle<v8::internal::FunctionTemplateInfo>, v8::internal::Handle<v8::internal::Object>, v8::internal::BuiltinArguments) [/usr/local/bin/node]
13: v8::internal::Builtin_Impl_HandleApiCall(v8::internal::BuiltinArguments, v8::internal::Isolate*) [/usr/local/bin/node]
14: 0x2eeaea8842fd
[1]    23639 abort      npm run build:prod-pi
```

검색을 좀해보았더니 그런 문제는 아니었던거 같았고

> $ node --max_old_space_size=5048 ./node_modules/@angular/cli/bin/ng build --prod --aot --no-sourcemap

기존의 `ng build --prod --aot` 빌드 스크립트를 위와 같이 바꾸면 빌드가 성공한다. 청크 파일들도 정상적으로 생성된다.

노드의 자바스크립트 힙 메모리를 OS 기본설정을 넘어서기 때문에 발생하는 문제였는데
-> 강제로 할당량을 늘려서 프로세스에 할당함으로써 해결하는 것.

-
Referencing
: [https://stackoverflow.com/questions/43369378/javascript-heap-out-of-memory-error-while-creating-build-in-angular-2/43389091?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa](https://stackoverflow.com/questions/43369378/javascript-heap-out-of-memory-error-while-creating-build-in-angular-2/43389091?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)
: [http://geeklearning.io/angular-aot-webpack-memory-trick/](http://geeklearning.io/angular-aot-webpack-memory-trick/)
