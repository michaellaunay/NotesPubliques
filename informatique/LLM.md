# Procédure de Vincent

[https://huggingface.co/TheBloke/Mixtral-8x7B-Instruct-v0.1-GGUF/tree/main](https://huggingface.co/TheBloke/Mixtral-8x7B-Instruct-v0.1-GGUF/tree/main)
# [https://huggingface.co/TheBloke/Mixtral-8x7B-Instruct-v0.1-GGUF/blob/main/README.md](https://huggingface.co/TheBloke/Mixtral-8x7B-Instruct-v0.1-GGUF/blob/main/README.md)
# [https://github.com/ggerganov/llama.cpp/pull/4406#issuecomment-1858988365](https://github.com/ggerganov/llama.cpp/pull/4406#issuecomment-1858988365)
curl -LO [https://huggingface.co/TheBloke/Mixtral-8x7B-Instruct-v0.1-GGUF/resolve/main/mixtral-8x7b-instruct-v0.1.Q4_K_M.gguf?download=true](https://huggingface.co/TheBloke/Mixtral-8x7B-Instruct-v0.1-GGUF/resolve/main/mixtral-8x7b-instruct-v0.1.Q4_K_M.gguf?download=true)
# [https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF](https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF)
curl -LO [https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF/resolve/main/mistral-7b-instruct-v0.2.Q4_K_M.gguf?download=true](https://huggingface.co/TheBloke/Mistral-7B-Instruct-v0.2-GGUF/resolve/main/mistral-7b-instruct-v0.2.Q4_K_M.gguf?download=true)

sudo apt install nvidia-cuda-toolkit

# Build from llama.cpp
git clone git@github.com:ggerganov/llama.cpp.git
cd llama.cpp
#git checkout b1671 # 2023-12-21 12:55
mkdir build
cd build
cmake .. -DLLAMA_CUBLAS=ON #-DLLAMA_CUDA_FORCE_MMQ:BOOL=ON
cmake --build . --config Release

# [https://github.com/ggerganov/llama.cpp/pull/4406](https://github.com/ggerganov/llama.cpp/pull/4406)
cd bin
./main -m ~/mixtral-8x7b-instruct-v0.1.Q4_K_M.gguf -p "[INST] Write a Python function that downloads a file from a URL[/INST]" --color -c 2048 --temp 0.7 --repeat_penalty 1.1 -n -1 -ngl 8
# you can increase -ngl (--gpu-layers) if you can

You can look at the gpu memory with:
watch nvidia-smi

Run the server:
./server -m ~/mixtral-8x7b-instruct-v0.1.Q4_K_M.gguf -c 2048 -ngl 7

Mistral 7B:
./main -m ~/mistral-7b-instruct-v0.2.Q4_K_M.gguf -p "[INST] Write a Python function that downloads a file from a URL [/INST]" --color -c 2048 --temp 0.7 --repeat_penalty 1.1 -n -1 -ngl 33

JSON Schema / bnf Grammar / Guided generation
[https://github.com/ggerganov/llama.cpp/pull/1773](https://github.com/ggerganov/llama.cpp/pull/1773)

./main -m ~/mistral-7b-instruct-v0.2.Q4_K_M.gguf -p "[INST] Give me a random json of a list of users with name age gender. [/INST]" --color -c 2048 --temp 0.7 --repeat_penalty 1.1 -n -1 --grammar-file ../../grammars/json_arr.gbnf

[https://github.com/ggerganov/llama.cpp/blob/master/grammars/README.md](https://github.com/ggerganov/llama.cpp/blob/master/grammars/README.md)
[https://github.com/ggerganov/llama.cpp/issues/1300#issuecomment-1692531185](https://github.com/ggerganov/llama.cpp/issues/1300#issuecomment-1692531185)
For generating arbitrary JSON, there's a JSON grammar provided in grammars/json.gbnf:
For conforming to a JSON schema, there's examples/json-schema-to-grammar.py :
[https://github.com/ggerganov/llama.cpp/blob/master/examples/json-schema-to-grammar.py](https://github.com/ggerganov/llama.cpp/blob/master/examples/json-schema-to-grammar.py)
python3 examples/json-schema-to-grammar.py ~/tmp/student.json

Generate grammar from typescript types:
[https://grammar.intrinsiclabs.ai/](https://grammar.intrinsiclabs.ai/) [https://github.com/IntrinsicLabsAI/grammar-builder](https://github.com/IntrinsicLabsAI/grammar-builder)
[https://github.com/IntrinsicLabsAI/gbnfgen](https://github.com/IntrinsicLabsAI/gbnfgen)
[https://github.com/ggerganov/llama.cpp/discussions/2494](https://github.com/ggerganov/llama.cpp/discussions/2494)

[https://www.imaurer.com/llama-cpp-grammars/](https://www.imaurer.com/llama-cpp-grammars/)
[https://til.simonwillison.net/llms/llama-cpp-python-grammars](https://til.simonwillison.net/llms/llama-cpp-python-grammars)

[https://github.com/3DStreet/text-to-street/blob/main/functions/texttostreetdemo.py](https://github.com/3DStreet/text-to-street/blob/main/functions/texttostreetdemo.py)
./main -m ~/mistral-7b-instruct-v0.2.Q4_K_M.gguf -p "[INST] create a scene with a road and sidewalks [/INST]" --color -c 2048 --temp 0.7 --repeat_penalty 1.1 -n -1  --grammar "$(python3 ../../examples/json-schema-to-grammar.py ~/tmp/3dstreet.json)"

Si vous voulez utiliser le chat avec la commande llm dans le terminal :
[https://github.com/simonw/llm](https://github.com/simonw/llm)  
[https://github.com/simonw/llm-llama-cpp](https://github.com/simonw/llm-llama-cpp)  
Installation :  

python3 -m pip install --user pipx
python3 -m pipx ensurepath
pipx install llm # 0.13.1
llm install llm-llama-cpp # 0.3
CMAKE_ARGS="-DLLAMA_CUBLAS=on" FORCE_CMAKE=1 llm install llama-cpp-python # 0.2.33

llm llama-cpp add-model -a mistral ~/mistral-7b-instruct-v0.2.Q4_K_M.gguf
llm llama-cpp add-model -a mixtral ~/mixtral-8x7b-instruct-v0.1.Q4_K_M.gguf
llm llama-cpp models

Quelque commandes :  

llm -m mistral -o n_gpu_layers 33 -o n_ctx 2048 '[INST] Write a Python function that downloads a file from a URL [/INST]'
llm -m mixtral -o n_gpu_layers 28 -o n_ctx 2048 '[INST] Write a Python function that downloads a file from a URL [/INST]'

llm chat -m mistral -o n_gpu_layers 33 -o n_ctx 2048
llm chat -m mixtral -o n_gpu_layers 28 -o n_ctx 2048

n_gpu_layers 28 c'est si t'as un gpu avec 24 Go, si t'as 8 Go c'est plutôt n_gpu_layers 7 (modifié) 

![GitHub](https://slack-imgs.com/?c=1&o1=wi32.he32.si&url=https%3A%2F%2Fa.slack-edge.com%2F80588%2Fimg%2Funfurl_icons%2Fgithub.png)GitHub

[GitHub - simonw/llm: Access large language models from the command-line](https://github.com/simonw/llm)

Access large language models from the command-line - GitHub - simonw/llm: Access large language models from the command-line (132 ko)

[https://github.com/simonw/llm](https://github.com/simonw/llm "GitHub - simonw/llm: Access large language models from the command-line")

[](https://github.com/simonw/llm)

![GitHub](https://slack-imgs.com/?c=1&o1=wi32.he32.si&url=https%3A%2F%2Fa.slack-edge.com%2F80588%2Fimg%2Funfurl_icons%2Fgithub.png)GitHub

[GitHub - simonw/llm-llama-cpp: LLM plugin for running models using llama.cpp](https://github.com/simonw/llm-llama-cpp)

LLM plugin for running models using llama.cpp. Contribute to simonw/llm-llama-cpp development by creating an account on GitHub. (133 ko)