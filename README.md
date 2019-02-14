## Example 2: *Hello world*

To get started let's make a small addon which is the C++ equivalent of
the following JavaScript code:

```js
module.exports.hello = function() { return 'world'; };
```

### Step 1

First we need to set up an npm-compatible package. Use `npm init` in a new directory to create the skeleton *package.json*.

### Step 2

Next we need to install **[NAN](https://github.com/rvagg/nan)**. NAN will serve as a thin abstraction layer between the C++ code we write and the Node and V8 APIs so that we can target multiple versions of Node without worrying too much about the changing V8 or Node APIs. Run `npm install nan@latest --save` to install NAN and save it as a `"dependency"` in your *package.json*.

### Step 3

All Node addons are compiled using the **[GYP](http://code.google.com/p/gyp/wiki/GypUserDocumentation)** build tool via **[node-gyp](https://github.com/TooTallNate/node-gyp)**. Add a `"gypfile": true` entry to your *package.json* so that npm knows this is a binary addon that needs compiling and it needs to invoke node-gyp. When node-gyp is invoked, it looks for a *binding.gyp* file in the same directory as your *package.json*. This file is written in YAML and describes the particulars of your build, including the source files and any binary dependencies. node-gyp will invoke GYP with your *binding.gyp* and also the [common.gypi](https://github.com/joyent/node/blob/master/common.gypi) settings file found in the source of the version of Node it is invoked with. Because your code will be compiled against the Node source and it requires the common.gypi file for that source, it must also download the complete tarball of the particular release you are running. These tarballs are stored in ~/.node-gyp/ so they only need to be installed once.

Create a *binding.gyp* file with the following contents:

```yaml
{
  "targets": [
    {
      "target_name": "hello",
      "sources": [ "hello.cc" ],
      "include_dirs": [
        "<!(node -e \"require('nan')\")"
      ]
    }
  ]
}
```

This file tells GYP the following things:

 1. The binary output target is called "hello", we will end up with a "hello.node" file either in ./build/Release/ or ./build/Debug/ depending on how node-gyp is invoked and your environment variables.
 1. The only source file to compile is "hello.cc" in the same directory as the *binding.gyp*.
 1. When compiling, the directory containing NAN needs to be used as an "include", i.e. compile with -Ipath/to/NAN. The NAN path is found by executing the command `node -e "require('nan')"` (try running this from the command-line!).

### Step 4

Now write some code!
First we create a file `hello.cc` and the following contents:

```c++
#include <nan.h>

void Method(const Nan::FunctionCallbackInfo<v8::Value>& info) {
  info.GetReturnValue().Set(Nan::New("world").ToLocalChecked());
}

void Init(v8::Local<v8::Object> exports) {
  exports->Set(Nan::New("hello").ToLocalChecked(),
               Nan::New<v8::FunctionTemplate>(Method)->GetFunction());
}

NODE_MODULE(hello, Init)
```
### Step 5

Compile your addon: if you don't have `node-gyp` installed, use `sudo npm install node-gyp -g` to install it. It comes bundled with npm but is not normally linked as an executable in your `PATH` so it's best to install it separately.

Run `node-gyp configure` to set up the build fixtures. In the *./build/* directory, a *Makefile* and associated property files will be created on Unix systems and a *vcxproj* file will be created on Windows.

Next, run `node-gyp build` to start the build process and watch your addon compile. You can use `node-gyp build` again to incrementally recompile changed source files. Alternatively you can use `node-gyp rebuild` to combine the `configure` and `build` steps in one.

You should now have a compiled binary addon ready for use. Node will load it just like it loads a .js module file and include it in your running application. The binary file should be be located at *./build/Release/hello.node*.

### Step 6

Write some JavaScript!

Create a file called *hello.js* with the following contents:

```js
var addon = require('./build/Release/hello.node');

console.log(addon.hello());
```

The first line is responsible for loading our compiled add-on and pulling in the exports to our module. The path may substitute Debug for Release if you have run node-gyp with the `--debug` flag or you have used another method to signal to GYP that this should be a debug build.

The best approach to this uncertainty of where the module file is located is to pull in an additional dependency, **[node-bindings](https://github.com/TooTallNate/node-bindings)** which will locate the binary for you, use it like so:

```js
var addon = require('bindings')('hello.node')
```

The invocation of our addon comes from `console.log(addon.hello())` where we fetch the `Method` function in our C++ code and execute it. It returns a `"hello"` String which is printed to the console.

The [nan](./nan/) directory contains a complete copy of this example. Run `npm install` and `node hello.js` to compile and run the code.

The [node_0.10](./node_0.10/) directory contains a version of the example without NAN, compatible with Node 0.10 and prior (only).

The [node_0.12](./node_0.12/) directory contains a version of the example without NAN, compatible with Node 0.11 and above (only).

***[Proceed to example 2 &raquo;](../2_function_arguments/)***

***[Index](../#readme)***
