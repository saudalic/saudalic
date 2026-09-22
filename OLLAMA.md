# Running Claude Code against a local Ollama model

Claude Code talks to the Anthropic API by default. To point it at a locally
running [Ollama](https://ollama.com) model instead, you need an
Anthropic-API-compatible proxy in front of Ollama, since Ollama's native API
isn't wire-compatible with the Anthropic Messages API.

## 1. Install and start Ollama

```bash
# macOS
brew install ollama

# Linux
curl -fsSL https://ollama.com/install.sh | sh

ollama serve
```

## 2. Pull a model

```bash
ollama pull llama3.1
```

## 3. Run a translation proxy

[LiteLLM](https://github.com/BerriAI/litellm) can expose an Anthropic-compatible
`/v1/messages` endpoint backed by Ollama:

```bash
pip install 'litellm[proxy]'

cat > litellm-config.yaml <<'EOF'
model_list:
  - model_name: claude-3-5-sonnet-20241022
    litellm_params:
      model: ollama/llama3.1
      api_base: http://localhost:11434
EOF

litellm --config litellm-config.yaml --port 4000
```

## 4. Point Claude Code at the proxy

```bash
export ANTHROPIC_BASE_URL=http://localhost:4000
export ANTHROPIC_API_KEY=sk-anything   # LiteLLM ignores the value by default

claude
```

Claude Code will now route requests through LiteLLM to your local Ollama
model instead of the hosted Anthropic API.

## Notes

- Local models are generally far less capable than the hosted Claude models,
  especially at agentic tool use, so expect degraded results.
- Tool-calling support depends on the specific Ollama model; not all models
  handle Claude Code's tool-use protocol well.
- This setup is community-driven and unofficial; Anthropic does not support
  or test Claude Code against local models.
