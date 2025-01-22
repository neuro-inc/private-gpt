## Pre-requisites
* Apolo cli. [Instructions](https://docs.apolo.us/index/cli/installing)
* HuggingFace access to the model you want to deploy. [For example LLAMA](https://huggingface.co/meta-llama/Llama-3.1-8B-Instruct)

## Run on apolo / neu.ro platform
Note: this setup is mostly for POC purposes. For production-ready setup, you'll need to replace some of it's components with production-ready Apps.

1. `$ git clone` this repo && `$ cd` into root of it.
1. Build image for web app with `$ apolo-flow build privategpt`
2. Create block storage for PGVector with `$ apolo disk create --name pgdata 10G --timeout-unused 100d`
3. Create secret with HuggingFace token to pull models `$ apolo secret add HF_TOKEN <token>` (see https://huggingface.co/settings/tokens)
4. `$ apolo-flow run pgvector` -- start vector store
5. `$ apolo-flow run ollama` -- start embeddings server
6. `$ apolo-flow run vllm` -- start LLM inference server. Note: if you want to change LLM hosted there, change it in bash command and in `env.VLLM_MODEL` of `pgpt` job.
7. `$ apolo-flow run pgpt` -- start PrivateGPT web server.

### Running PrivateGPT as stand-alone job
<details>
<summary> Instruction </summary>

Currently, we support only deployment case with vLLM as LLM inference server, PGVector as a vector store and Ollama as embeddings server.

Use following environment variables to configure PrivateGPT running within the job:

Scheme: `env name (value type, required/optional) -- description`.

LLM config section:
- `VLLM_API_BASE` (URL, required) -- HTTP endpoint for LLM inference
- `VLLM_MODEL` (hugging face model reference, required) -- LLM model name to use (must be available at inference server).
- `VLLM_TOKENIZER` (hugging face model reference, required) -- tokenized to use while sending requests to LLM
- `VLLM_MAX_NEW_TOKENS` (int, required) -- controls the response size from LLM
- `VLLM_CONTEXT_WINDOW` (int, required) -- controls context size that will be sent to LLM
- `VLLM_TEMPERATURE` (float 0 < x < 1, optional) -- temperature parameter ('creativeness') for LLM. Less value -- more strict penalty for going out of provided context.

PGVector config section:
- `POSTGRES_HOST` (str, required) -- hostname for Postgres instance with PGVector installed
- `POSTGRES_PORT` (int, optional) -- TCP port for Postgres instance
- `POSTGRES_DB` (str, required) -- Postgres database name
- `POSTGRES_USER` (str, required) -- username for Postgres DB
- `POSTGRES_PASSWORD` (str, required) -- password for Postgres DB

Embeddings config section:
- `OLLAMA_API_BASE` (URL, required) -- Ollama server endpoint. Must be already running.
- `OLLAMA_EMBEDDING_MODEL` (str, optional) -- embeddings model to use. Must be already loaded into Ollama instance

Having above values, run job with
`$ apolo run --volume storage:.apps/pgpt/data:/home/worker/app/local_data --http-port=8080 ghcr.io/neuro-inc/private-gpt`

Other platform-related configurations like `--life-span`, etc. also work here.

</details>
