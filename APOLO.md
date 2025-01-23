## Pre-requisites
* Apolo cli. [Instructions](https://docs.apolo.us/index/cli/installing)
* HuggingFace access to the model you want to deploy. [For example DeepSeek](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-32B)

## Run on apolo platform
Note: this setup is mostly for POC purposes. For production-ready setup, you'll need to replace some of it's components with production-ready Apps.

1. `$ git clone` this repo && `$ cd` into root of it.
1. Build image for web app with `$ apolo-flow build privategpt`
2. Create secret with HuggingFace token to pull models `$ apolo secret add HF_TOKEN <token>` (see https://huggingface.co/settings/tokens)
3. `$ apolo-flow run pgvector` -- start vector store
4. `$ apolo-flow run tei` -- start embeddings server
5. `$ apolo-flow run vllm` -- start LLM inference server. Note: if you want to change LLM hosted there, change it in `live.yaml:defaults.env.VLLM_MODEL`.
6. `$ apolo-flow run pgpt` -- start PrivateGPT web server.

### Running PrivateGPT as stand-alone job
<details>
<summary> Instruction </summary>

Currently, we support only deployment case with vLLM as LLM inference server, PGVector as a vector store and TextEmbeddingsInference as embeddings server.

Use following environment variables to configure PrivateGPT instance:

Scheme: `env name (value type, required/optional) -- description`.

Shared among all the jobs:
- `VLLM_MODEL` (hugging face model reference, required) -- LLM model name to use (must be available at inference server).
- `VLLM_TOKENIZER` (hugging face model reference, required) -- tokenized to use while sending requests to LLM
- `VLLM_CONTEXT_WINDOW` (int, required) -- controls context size that will be sent to LLM

LLM config section:
- `VLLM_API_BASE` (URL, required) -- HTTP endpoint for LLM inference
- `VLLM_TEMPERATURE` (float 0 < x < 1, optional) -- temperature parameter ('creativeness') for LLM. Less value -- more strict penalty for going out of provided context.

Embeddings config section:
- `EMBEDDING_MODEL` (str, optional) -- embeddings model to use

Other platform-related configurations like `--life-span`, etc. also work here.

</details>
