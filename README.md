## wasi-threads exporation
This project contains code for exploring wasi-threads specifically trying to
get ggml and llama.cpp working with wasi-threads using the WASI-SDK.

This project is currently using make and cmake as I don't really know which
the upstream project will use and I want to be able to test both.

### Update
I started investigating this by first trying out wasi-threads to make sure that
worked, followed by compiling ggml to wasm32-wasi. After that I tried to get
llama.cpp compiled to wasm32-wasi. There were a few issue that had to be worked
around, for example c++ exception that are used in llama.cpp and that is
something what would have to be sorted out some way.

But the main issue was that after seeing a number of wasm example around, which
most I think use wasi-nn, I did not consider the 4GB memory limit that is
imposed by the wasm32-wasi target which has to do with 32bit addresses. Most of
the models are larger than that after beeing loaded which will cause out of
memory errors. The example in this project currently uses `llama-2-7b.Q2_K.gguf`
which is under 4GB at runtime and that is why it works. But this is just a base
model a chat model would be larger and would probably not work. We would need
support for [wasm64-wasi] for this to work with larger models.

*So how can the wasi-nn examples that I have seen work with larger model sizes
then?*    
Well I believe the answer to that is that they are extensions/plugins/part of
the wasm runtime implementation used, like wasmtime or wasmedge. So the users
code interacts with the wasi-nn API but the actual implementation is not part of
users wasm code. The implementation for wasmedge is called a plugin and the
runtime does not have the memory restrictions that wasm32 does which is the
reason this works.

*So is the answer to just use wasi-nn?*  
That would work but I think but there might be value in having a pure wasm
implementation for llama.cpp like this project is trying to do. The reason for
this is with the fast pace of development and research, which llama.cpp is very
fast at adopting, for these updates to reach the end user the wasm runtime
will needed to apply those updates, release a new version, and the end user
would also need to update their environment to use that new version. Perhaps
there is room for both approaches?


### Setup/Configuration
This project requires wasi-sdk to be installed:
```console
$ make clone-wasi-sdk build-wasi-sdk extract-wasi-sdk
```

And wasmtime is the WASI runtime that is used and is also required:
```console
$ make clone-wasmtime build-wasmtime
```

We also need a model to be able to run which can be downloaded by the following
command:
```console
$ make download-model
```
This will download llama-2-7b-chat.Q4_0.gguf from huggingface.

### Building llama.cpp with wasi-threads support
The llama.cpp library is included as a git submodule so make sure to run the
following command if it is not already cloned:
```console
$ git submodule update --init --recursive
```

The following command will build llama.cpp
```console
$ make build-llama-wasi
```
This part is still under development and I'm not exactly sure what and where
the changes to llama.cpp should end up. 
TODO: figure out what the best approach is for these changes.

### Building
With the above setup/configuration/build it should be possible to build this
project:
```console
$ make out/wasi-threads.wasm
```
The above will use the [make](./Makefile) file to build wasi-threads.wasm.

This can also be built using [cmake](./CMakeLists.txt):
```console
$ make cmake-build
```

### Running
The following command will run the main.wasm module if the project was built
using make::
```console
$ make run
env WASMTIME_BACKTRACE_DETAILS=1 /home/danielbevenius/work/c++/wasi-threads/wasmtime/target/release/wasmtime run  -W threads -S threads \
        --dir ./models out/wasi-threads.wasm -- \
models/llama-2-7b.Q2_K.gguf "What is LoRA?"
llama.cpp example using model: models/llama-2-7b.Q2_K.gguf
llama_file: opening models/llama-2-7b.Q2_K.gguf
llama_model_loader: loaded meta data with 19 key-value pairs and 291 tensors from models/llama-2-7b.Q2_K.gguf (version GGUF V2)
llama_model_loader: - tensor    0:                token_embd.weight q2_K     [  4096, 32000,     1,     1 ]
llama_model_loader: - tensor    1:           blk.0.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor    2:            blk.0.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor    3:            blk.0.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor    4:              blk.0.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor    5:            blk.0.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor    6:              blk.0.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor    7:         blk.0.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor    8:              blk.0.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor    9:              blk.0.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   10:           blk.1.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   11:            blk.1.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor   12:            blk.1.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   13:              blk.1.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   14:            blk.1.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   15:              blk.1.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   16:         blk.1.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   17:              blk.1.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   18:              blk.1.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   19:          blk.10.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   20:           blk.10.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor   21:           blk.10.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   22:             blk.10.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   23:           blk.10.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   24:             blk.10.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   25:        blk.10.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   26:             blk.10.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   27:             blk.10.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   28:          blk.11.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   29:           blk.11.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor   30:           blk.11.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   31:             blk.11.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   32:           blk.11.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   33:             blk.11.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   34:        blk.11.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   35:             blk.11.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   36:             blk.11.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   37:          blk.12.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   38:           blk.12.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor   39:           blk.12.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   40:             blk.12.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   41:           blk.12.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   42:             blk.12.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   43:        blk.12.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   44:             blk.12.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   45:             blk.12.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   46:          blk.13.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   47:           blk.13.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor   48:           blk.13.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   49:             blk.13.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   50:           blk.13.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   51:             blk.13.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   52:        blk.13.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   53:             blk.13.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   54:             blk.13.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   55:          blk.14.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   56:           blk.14.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor   57:           blk.14.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   58:             blk.14.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   59:           blk.14.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   60:             blk.14.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   61:        blk.14.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   62:             blk.14.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   63:             blk.14.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   64:          blk.15.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   65:           blk.15.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor   66:           blk.15.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   67:             blk.15.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   68:           blk.15.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   69:             blk.15.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   70:        blk.15.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   71:             blk.15.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   72:             blk.15.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   73:          blk.16.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   74:           blk.16.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor   75:           blk.16.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   76:             blk.16.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   77:           blk.16.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   78:             blk.16.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   79:        blk.16.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   80:             blk.16.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   81:             blk.16.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   82:          blk.17.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   83:           blk.17.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor   84:           blk.17.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   85:             blk.17.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   86:           blk.17.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   87:             blk.17.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   88:        blk.17.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   89:             blk.17.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   90:             blk.17.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   91:          blk.18.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   92:           blk.18.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor   93:           blk.18.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   94:             blk.18.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor   95:           blk.18.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor   96:             blk.18.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   97:        blk.18.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   98:             blk.18.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor   99:             blk.18.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  100:          blk.19.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  101:           blk.19.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  102:           blk.19.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  103:             blk.19.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  104:           blk.19.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  105:             blk.19.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  106:        blk.19.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  107:             blk.19.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  108:             blk.19.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  109:           blk.2.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  110:            blk.2.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  111:            blk.2.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  112:              blk.2.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  113:            blk.2.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  114:              blk.2.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  115:         blk.2.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  116:              blk.2.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  117:              blk.2.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  118:          blk.20.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  119:           blk.20.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  120:           blk.20.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  121:             blk.20.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  122:           blk.20.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  123:             blk.20.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  124:        blk.20.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  125:             blk.20.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  126:             blk.20.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  127:          blk.21.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  128:           blk.21.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  129:           blk.21.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  130:             blk.21.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  131:           blk.21.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  132:             blk.21.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  133:        blk.21.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  134:             blk.21.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  135:             blk.21.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  136:          blk.22.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  137:           blk.22.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  138:           blk.22.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  139:             blk.22.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  140:           blk.22.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  141:             blk.22.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  142:        blk.22.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  143:             blk.22.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  144:             blk.22.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  145:          blk.23.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  146:           blk.23.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  147:           blk.23.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  148:             blk.23.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  149:           blk.23.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  150:             blk.23.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  151:        blk.23.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  152:             blk.23.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  153:             blk.23.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  154:           blk.3.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  155:            blk.3.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  156:            blk.3.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  157:              blk.3.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  158:            blk.3.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  159:              blk.3.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  160:         blk.3.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  161:              blk.3.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  162:              blk.3.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  163:           blk.4.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  164:            blk.4.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  165:            blk.4.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  166:              blk.4.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  167:            blk.4.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  168:              blk.4.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  169:         blk.4.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  170:              blk.4.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  171:              blk.4.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  172:           blk.5.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  173:            blk.5.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  174:            blk.5.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  175:              blk.5.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  176:            blk.5.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  177:              blk.5.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  178:         blk.5.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  179:              blk.5.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  180:              blk.5.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  181:           blk.6.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  182:            blk.6.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  183:            blk.6.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  184:              blk.6.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  185:            blk.6.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  186:              blk.6.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  187:         blk.6.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  188:              blk.6.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  189:              blk.6.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  190:           blk.7.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  191:            blk.7.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  192:            blk.7.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  193:              blk.7.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  194:            blk.7.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  195:              blk.7.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  196:         blk.7.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  197:              blk.7.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  198:              blk.7.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  199:           blk.8.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  200:            blk.8.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  201:            blk.8.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  202:              blk.8.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  203:            blk.8.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  204:              blk.8.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  205:         blk.8.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  206:              blk.8.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  207:              blk.8.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  208:           blk.9.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  209:            blk.9.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  210:            blk.9.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  211:              blk.9.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  212:            blk.9.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  213:              blk.9.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  214:         blk.9.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  215:              blk.9.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  216:              blk.9.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  217:                    output.weight q6_K     [  4096, 32000,     1,     1 ]
llama_model_loader: - tensor  218:          blk.24.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  219:           blk.24.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  220:           blk.24.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  221:             blk.24.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  222:           blk.24.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  223:             blk.24.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  224:        blk.24.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  225:             blk.24.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  226:             blk.24.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  227:          blk.25.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  228:           blk.25.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  229:           blk.25.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  230:             blk.25.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  231:           blk.25.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  232:             blk.25.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  233:        blk.25.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  234:             blk.25.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  235:             blk.25.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  236:          blk.26.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  237:           blk.26.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  238:           blk.26.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  239:             blk.26.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  240:           blk.26.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  241:             blk.26.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  242:        blk.26.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  243:             blk.26.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  244:             blk.26.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  245:          blk.27.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  246:           blk.27.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  247:           blk.27.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  248:             blk.27.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  249:           blk.27.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  250:             blk.27.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  251:        blk.27.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  252:             blk.27.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  253:             blk.27.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  254:          blk.28.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  255:           blk.28.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  256:           blk.28.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  257:             blk.28.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  258:           blk.28.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  259:             blk.28.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  260:        blk.28.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  261:             blk.28.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  262:             blk.28.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  263:          blk.29.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  264:           blk.29.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  265:           blk.29.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  266:             blk.29.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  267:           blk.29.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  268:             blk.29.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  269:        blk.29.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  270:             blk.29.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  271:             blk.29.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  272:          blk.30.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  273:           blk.30.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  274:           blk.30.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  275:             blk.30.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  276:           blk.30.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  277:             blk.30.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  278:        blk.30.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  279:             blk.30.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  280:             blk.30.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  281:          blk.31.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  282:           blk.31.ffn_down.weight q3_K     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  283:           blk.31.ffn_gate.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  284:             blk.31.ffn_up.weight q3_K     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  285:           blk.31.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  286:             blk.31.attn_k.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  287:        blk.31.attn_output.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  288:             blk.31.attn_q.weight q2_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  289:             blk.31.attn_v.weight q3_K     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  290:               output_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
llama_model_loader: - kv   0:                       general.architecture str              = llama
llama_model_loader: - kv   1:                               general.name str              = LLaMA v2
llama_model_loader: - kv   2:                       llama.context_length u32              = 4096
llama_model_loader: - kv   3:                     llama.embedding_length u32              = 4096
llama_model_loader: - kv   4:                          llama.block_count u32              = 32
llama_model_loader: - kv   5:                  llama.feed_forward_length u32              = 11008
llama_model_loader: - kv   6:                 llama.rope.dimension_count u32              = 128
llama_model_loader: - kv   7:                 llama.attention.head_count u32              = 32
llama_model_loader: - kv   8:              llama.attention.head_count_kv u32              = 32
llama_model_loader: - kv   9:     llama.attention.layer_norm_rms_epsilon f32              = 0.000010
llama_model_loader: - kv  10:                          general.file_type u32              = 10
llama_model_loader: - kv  11:                       tokenizer.ggml.model str              = llama
llama_model_loader: - kv  12:                      tokenizer.ggml.tokens arr[str,32000]   = ["<unk>", "<s>", "</s>", "<0x00>", "<...
llama_model_loader: - kv  13:                      tokenizer.ggml.scores arr[f32,32000]   = [0.000000, 0.000000, 0.000000, 0.0000...
llama_model_loader: - kv  14:                  tokenizer.ggml.token_type arr[i32,32000]   = [2, 3, 3, 6, 6, 6, 6, 6, 6, 6, 6, 6, ...
llama_model_loader: - kv  15:                tokenizer.ggml.bos_token_id u32              = 1
llama_model_loader: - kv  16:                tokenizer.ggml.eos_token_id u32              = 2
llama_model_loader: - kv  17:            tokenizer.ggml.unknown_token_id u32              = 0
llama_model_loader: - kv  18:               general.quantization_version u32              = 2
llama_model_loader: - type  f32:   65 tensors
llama_model_loader: - type q2_K:   65 tensors
llama_model_loader: - type q3_K:  160 tensors
llama_model_loader: - type q6_K:    1 tensors
llama_model_loader: mmap is not supported on this platform
llm_load_vocab: special tokens definition check successful ( 259/32000 ).
llm_load_print_meta: format           = GGUF V2
llm_load_print_meta: arch             = llama
llm_load_print_meta: vocab type       = SPM
llm_load_print_meta: n_vocab          = 32000
llm_load_print_meta: n_merges         = 0
llm_load_print_meta: n_ctx_train      = 4096
llm_load_print_meta: n_embd           = 4096
llm_load_print_meta: n_head           = 32
llm_load_print_meta: n_head_kv        = 32
llm_load_print_meta: n_layer          = 32
llm_load_print_meta: n_rot            = 128
llm_load_print_meta: n_gqa            = 1
llm_load_print_meta: f_norm_eps       = 0.0e+00
llm_load_print_meta: f_norm_rms_eps   = 1.0e-05
llm_load_print_meta: f_clamp_kqv      = 0.0e+00
llm_load_print_meta: f_max_alibi_bias = 0.0e+00
llm_load_print_meta: n_ff             = 11008
llm_load_print_meta: n_expert         = 0
llm_load_print_meta: n_expert_used    = 0
llm_load_print_meta: rope scaling     = linear
llm_load_print_meta: freq_base_train  = 10000.0
llm_load_print_meta: freq_scale_train = 1
llm_load_print_meta: n_yarn_orig_ctx  = 4096
llm_load_print_meta: rope_finetuned   = unknown
llm_load_print_meta: model type       = 7B
llm_load_print_meta: model ftype      = Q2_K
llm_load_print_meta: model params     = 6.74 B
llm_load_print_meta: model size       = 2.63 GiB (3.35 BPW) 
llm_load_print_meta: general.name     = LLaMA v2
llm_load_print_meta: BOS token        = 1 '<s>'
llm_load_print_meta: EOS token        = 2 '</s>'
llm_load_print_meta: UNK token        = 0 '<unk>'
llm_load_print_meta: LF token         = 13 '<0x0A>'
llm_load_tensors: ggml ctx size = 2694.41 MiB
llm_load_tensors: mem required  = 2694.41 MiB
.................................................................................................
llama_new_context_with_model: n_ctx      = 1024
llama_new_context_with_model: freq_base  = 10000.0
llama_new_context_with_model: freq_scale = 1
llama_new_context_with_model: KV self size  =  512.00 MiB, K (f16):  256.00 MiB, V (f16):  256.00 MiB
llama_build_graph: non-view tensors processed: 676/676
llama_new_context_with_model: compute buffer total size = 92.44 MiB

prompt: What is LoRA?
 Unterscheidung von LoRA und LoRa
 LoRa-Gateway
 LoRa-Gateway-App
 LoRa-Gateway-App-App
 LoRa-Gateway-App-App-App
 LoRa-Gateway-App-App-App-App
 LoRa-

Decoded 80 tokens
```
So this model does not really work for this kind of prompt. We would require a
larger model for this and we run into the memory issue mentioned above.

_work in progress..._

And use the following if it was build using cmake:
```console
$ make cmake-build-run
```

### Misc
The maximum memory size allowed is 4294967296, which is 4096 MiB. If this
is exceeded you will get an error like this:
```console
wasm-ld: error: maximum memory too large, cannot be greater than 4294967296
```
So the model that we choose must be able to fit within this limit. The one that
I initially used was llama-2-7b-chat.Q4_0.gguf which somewhere above 5 GiB. This
is a limitation of wasm32 which can only support 32-bit addresses and for this
to work we need to use 64-bit addresses. This will be supported by
[wasm64-wasi].

[wasm64-wasi]: https://github.com/WebAssembly/wasi-sdk/issues/185

