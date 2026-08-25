# Confidential LoRA Serving Demo

A [Tinfoil Container](https://docs.tinfoil.sh/containers/overview) serving
[openai/gpt-oss-20b](https://huggingface.co/openai/gpt-oss-20b) together with a
LoRA fine-tune ([text-to-SQL adapter](https://huggingface.co/jeeejeee/gpt-oss-20b-lora-adapter-text2sql))
on one attested endpoint, using a stock upstream
[vLLM](https://github.com/vllm-project/vllm) image pinned by digest.

Both the base weights and the adapter are pinned to exact Hugging Face
commits and mounted as read-only dm-verity model packs, so the enclave
attestation covers precisely which base model *and which fine-tune* are
serving — swapping either changes the measurement.

## Call it

The endpoint exposes an OpenAI-compatible API. The base model and the
fine-tune are separate model names on the same deployment:

```bash
# List models — shows the base and the adapter
curl https://<deployment-domain>/v1/models

# Base model
curl https://<deployment-domain>/v1/chat/completions \
  -H "Authorization: Bearer $TINFOIL_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"model": "gpt-oss-20b", "messages": [{"role": "user", "content": "..."}]}'

# LoRA fine-tune
curl https://<deployment-domain>/v1/chat/completions \
  -H "Authorization: Bearer $TINFOIL_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"model": "gpt-oss-20b-text2sql", "messages": [{"role": "user", "content": "..."}]}'
```

## Verify it

```bash
tinfoil attestation verify -e <deployment-domain> -r tinfoilsh/confidential-lora-demo
```

This checks the hardware attestation (Intel TDX + NVIDIA GPU) against the
measurements published for this repository's release, which bind the exact
`tinfoil-config.yml` above — image digest, base model hash, and adapter
hash included.

## Deploy your own

This repository is the entire deployment: edit `tinfoil-config.yml`, point
`models:` at your own base weights and adapters, push a tag, and deploy
from the [dashboard](https://dash.tinfoil.sh) or the `tinfoil` CLI. See the
[containers documentation](https://docs.tinfoil.sh/containers/overview).
