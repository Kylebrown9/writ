# WRIT (WASI Reactor Interface Tester)

A CLI tool that given a WIT specification, will correctly interpret and cast arguments for an arbitrarily specified function in a Wasm module, and run it.  In particular, this tool will be useful when writing and testing “reactor” style WASI modules, where there is no “main” routine to invoke.

When no WIT file is provided, the arguments will be interpreted as basic types the same way "wasmtime --invoke" works

To facilitate expression of complex types, the tool will accept JSON notation as input and produce JSON notation as output.

## Prerequisites
1. Download and install [Wasmtime](https://wasmtime.dev/)


2. Install [wit-bindgen](https://github.com/bytecodealliance/wit-bindgen). You might need to install [cargo](https://doc.rust-lang.org/cargo/getting-started/installation.html) first.

3. Download and install the [WASI SDK](https://github.com/WebAssembly/wasi-sdk/releases/tag/wasi-sdk-14)

## Installation
Clone this repository:
```sh
git clone https://github.com/singlestore-labs/writ.git
cd writ
```

## Usage
```console
$ python3 writ.py --help
usage: writ.py [-h] [-b [BINDING_PATH]] [-w [WIT_PATH]] ...

WASI Reactor Interface Tester

positional arguments:
  input_args            path to the wasm file, function name, input in json format

optional arguments:
  -h, --help            show this help message and exit
  -b [BINDING_PATH], --bindings [BINDING_PATH]
                        path to the binding folder
  -w [WIT_PATH], --wit [WIT_PATH]
                        path to the wit file
```

## Examples 
There are a few examples in `/test` directory. Running from `writ` base directory.

1. `power` example: 
```sh
python3 writ.py --wit test/int/power.wit test/int/power.wasm power-of 2 3
python3 writ.py test/int/power.wasm power-of 2 3
```
Output:
```console
8
```

2. `split string` example:
```sh
python3 writ.py --wit test/string/split.wit test/string/split.wasm split-str '"wasm_rocks_the_house"' '"_"'
```
Output:
```console
[{"str": "wasm", "idx": 0}, {"str": "rocks", "idx": 5}, {"str": "the", "idx": 11}, {"str": "house", "idx": 15}]
```

3. Same `power` example, but for float type
float:
```sh
python3 writ.py --wit test/float/power.wit test/int/power.wasm power-of 2 3
python3 writ.py test/float/power.wasm power-of 2 3
```
Output:
```console
8.0000
```
4. a machine learning model `sentiment analysis` example:
```
python3 writ.py --wit test/sentiment/sentiment.wit  test/sentiment/sentiment.wasm sentiment  '"have_a_nice_day"' '"_"'
```
Output:
```console
{"compound": 0.0, "positive": 0.0, "negative": 0.0, "neutral": 1.0}
```

5. `record` examples:
* Construct a struct `bar` giving a `string name` and an `integer age`:
```sh
python3 writ.py --wit test/record/record.wit test/record/record.wasm construct-bar '"meow"' 22
```
Output:
```console
{"name": "meow", "age": 22}
```
* Apply a function on a given struct (`bar`)
```sh
 python3 writ.py --wit test/record/record.wit test/record/record.wasm bar '{"name": "meow", "age": 22}'
```
Output:
```console
{"name": "meow", "age": 32}
```

* Construct a nested struct given an input
```sh
python3 writ.py --wit test/record/record.wit test/record/record.wasm deeper-bar '{"name": "meow", "age": 22}'
```
Output:
```console
{"id": 2, "x": {"id": 1, "x": {"name": "meow", "age": 32}}}
```

* Apply a function on a nested struct
```sh
 python3 writ.py --wit test/record/record.wit test/record/record.wasm rev-deeper-bar '{"id": 2, "x": {"id": 1, "x": {"name": "meow", "age": 32}}}'
```
Output:
```console
{"id": 4, "x": {"id": 1, "x": {"name": "meow", "age": 32}}}
```

6. Apply a function on a list of records (or any types that can be presented as JSON)
```sh
python3 writ.py --wit test/list_record/list_record.wit test/list_record/list_record.wasm test-list-record '[{"name": "doggo", "age": 42}, {"name":"meow", "age":28}]'
```
Output
```console
70
```
7. Apply a function on binary type

Notice: Bytes has to be presented as an array of integer, each integer represent one byte
```sh
python3 writ.py --wit test/hilbert/hilbert.wit test/hilbert/hilbert.wasm hilbert-encode '{"vec": [19,2,20,56,6,2,25,19], "min-value": 1.0, "max-value": 3.0, "scale": 6.0}'
```
Output:
```console
[{"idx": "0"}]
```
