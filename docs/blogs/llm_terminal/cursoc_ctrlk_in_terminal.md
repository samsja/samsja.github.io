# Local alternative to cursor ctrl k 

##

Using uv 
```
uv tool install llm
```

or pip 

```
pip install llm
```


install llm-gguf

```
llm install llm-gguf
```

Download qwen family

```
llm gguf download-model https://huggingface.co/Qwen/Qwen2.5-Coder-0.5B-Instruct-GGUF/resolve/main/qwen2.5-coder-0.5b-instruct-fp16.gguf --alias qwen-coder-2.5-0.5b --alias qw0.5
```

```
llm gguf download-model https://huggingface.co/Qwen/Qwen2.5-Coder-1.5B-Instruct-GGUF/resolve/main/qwen2.5-coder-1.5b-instruct-q8_0.gguf --alias qwen-coder-2.5-1.5b --alias qw1.5b
```

```
llm gguf download-model https://huggingface.co/Qwen/Qwen2.5-Coder-3B-Instruct-GGUF/resolve/main/qwen2.5-coder-3b-instruct-q8_0.gguf --alias qwen-coder-2.5-3b --alias qw3b
```

```
llm gguf download-model https://huggingface.co/Qwen/Qwen2.5-Coder-7B-Instruct-GGUF/resolve/main/qwen2.5-coder-7b-instruct-q8_0.gguf --alias qwen-coder-2.5-7b --alias qw7b
```


```
llm --system 'reply with ubuntu terminal commands only, no extra information' --model qw0.5 --save cmd
```


```
alias llmk='f() { llm -t cmd "\"$*\""; }; f'
```



