---
title: "Ollama를 서버에서 실행시켜보자"
excerpt: "공짜가 최고야"

categories:
  - "LLM"
tags:
  - ['llm', 'ollama']

permalink: /categories/dev/llm/ollama

toc: true
toc_sticky: true

date: 2024-10-23
last_modified_at: 2024-10-23
---

# 원격으로 다운을 못 받는 물리 서버에서 Ollama를 설치해보자

- 회사에서 개발을 하다보면 방화벽 때문에 원격 서버 파일 다운로드를 못하는 경험을 한번쯤은 했을 것이다
- 일일히 로컬에서 다운받아 `scp` 명령어로 옮겨야 돼서 매우 귀찮다
- 이번에 llm 프레임워크 ollama를 네트워크가 안되는 물리서버에 설치한 경험을 얘기해본다

## Ollama install

### 설치 파일 다운로드

```shell
curl -fsSL https://github.com/ollama/ollama/releases/download/v0.3.12/ollama-linux-amd64.tgz
sudo tar -C /usr -xzf ollama-linux-amd64.tgz
```

### 실행 파일

```shell
export OLLAMA_MODELS=/모델설치파일/ollama_models/
export OLLAMA_HOST=0.0.0.0
nohup ollama serve > ollama.log 2>&1 &
```

**환경 설정**
- OLLAMA_MODELS: 모델 설치파일 주소. 서버 내에 용량이 큰 디렉토리 주소로 설정하는 것이 좋다
- OLLAMA_HOST: 위와 같이 설정해줘야 `11434` 포트로 원격에서 요청이 가능하다 

## LLM 다운로드

### huggingface model download

**Python 코드**

- 아래를 실행시키면 `sample` 디렉토리로 다운받는다

```
from huggingface_hub import snapshot_download

model_id="레퍼지토리/모델명" 
snapshot_download(repo_id=model_id, local_dir="sample", local_dir_use_symlinks=False
```

### Model To GGUF

**gguf 변환 파일 다운**

```shell
$ git clone https://github.com/ggerganov/llama.cpp.git
```

**gguf 변환**

```shell
$ python llama.cpp/convert_hf_to_gguf.py sample --outfile 모델명.gguf --outtype q8_0
$ echo "from 모델명" > ./Modelfile
```

**ollama 모델 설치**

- 설치할 때 `Modelfile` 경로에 `gguf` 파일도 포함해야한다

```shell
$ ollama create 모델명 -f ./Modelfile
```

- 만약에 `open-webui`를 사용한다면 재시작해서 적용

**Modelfile 수정**

- 만약에 모델 추론 결과가 이상하다면 Modelfile을 알맞게 수정해야한다
- 난 해당 [llama-3.2-korean-bllossom-3b](https://huggingface.co/Bllossom/llama-3.2-Korean-Bllossom-3B) 모델을 사용하고 있었는데 외계어를 뱉는 것처럼 이상했다
- 아래와 같이 수정을 해서 다시 설치했더니 제대로 나왔다

```shell
from llama3.2-3b.gguf

SYSTEM """당신은 유용한 AI 어시스턴트입니다. 사용자의 질의에 대해 친절하고 정확하게 답변해야 합니다. You are a helpful AI assistant, you'll need to answer users' queries in a friendly and accurate manner. 모든 대답은 한국어(Korean)으로 대답해주세요."""

TEMPLATE """{{- if .System }}
<s>{{ .System }}</s>
{{- end }}
<s>Human:
{{ .Prompt }}</s>
<s>Assistant:
"""

PARAMETER temperature 0.6
PARAMETER num_predict 3000
PARAMETER num_ctx 4096
PARAMETER stop <s>
PARAMETER stop </s>
PARAMETER stop <|eot_id|>
```




