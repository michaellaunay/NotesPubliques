Nous allons suivre la documentation https://platform.openai.com/docs/api-reference/authentication
Créer une [clé](https://platform.openai.com/api-keys) applicative (ici python_test) copier cette clé dans un fichier .env (vérifiez bien vos règles git.ignore pour systématiquement exclure ce type de fichier d'une éventuelle publication)


```bash
michaellaunay@leojag:~/workspace$ export OPENAI_API_KEY="YOUR_KEY"
michaellaunay@leojag:~/workspace$ mkdir openai
michaellaunay@leojag:~/workspace$ python3 -m venv openai
michaellaunay@leojag:~/workspace$ cd openai/
michaellaunay@leojag:~/workspace/openai$ source bin/activate
(openai) michaellaunay@leojag:~/workspace/openai$ pip install openai
Collecting openai
  Downloading openai-1.14.3-py3-none-any.whl (262 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 262.9/262.9 KB 5.2 MB/s eta 0:00:00
...
Successfully installed annotated-types-0.6.0 anyio-4.3.0 certifi-2024.2.2 distro-1.9.0 exceptiongroup-1.2.0 h11-0.14.0 httpcore-1.0.5 httpx-0.27.0 idna-3.6 openai-1.14.3 pydantic-2.6.4 pydantic-core-2.16.3 sniffio-1.3.1 tqdm-4.66.2 typing-extensions-4.10.0

```
Ajoutons un crédit (ici 10$)
Vérifions que cela fonctionne :
```bash
michaellaunay@leojag:~$ curl https://api.openai.com/v1/chat/completions   -H "Content-Type: application/json"   -H "Authorization: Bearer $OPENAI_API_KEY"   -d '{
     "model": "gpt-3.5-turbo",
     "messages": [{"role": "user", "content": "Say this is a test!"}],
     "temperature": 0.7
   }'
{
  "id": "chatcmpl-97VCm43YfAmDalBiWOzJ6kRMWlYn4",
  "object": "chat.completion",
  "created": 1711575276,
  "model": "gpt-3.5-turbo-0125",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "This is a test!"
      },
      "logprobs": null,
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 13,
    "completion_tokens": 5,
    "total_tokens": 18
  },
  "system_fingerprint": "fp_3bc1b5746c"
}

```

Essayer avec python
```python
from openai import OpenAI

client = OpenAI()

stream = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Say this is a test"}],
    stream=True,
)
for chunk in stream:
    if chunk.choices[0].delta.content is not None:
        print(chunk.choices[0].delta.content, end="")
```
Le résultat est
```bash
michaellaunay@leojag:~/workspace/openai/src$ source ../bin/activate
(openai) michaellaunay@leojag:~/workspace/openai/src$ python3 test_stream.py 
This is a test.(openai)
```

Nous pourrons maintenant avoir accès au fine-tuning, à l'embeddings etc