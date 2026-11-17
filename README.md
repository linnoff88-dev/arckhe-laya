# Laya INEMA â€” triagem prÃ¡tica em portuguÃªs

**ðŸ‡§ðŸ‡· [PortuguÃªs](README.md) Â· ðŸ‡ºðŸ‡¸ [English](README.en.md) Â· ðŸ‡ªðŸ‡¸ [EspaÃ±ol](README.es.md)**

AdaptaÃ§Ã£o do [Laya, da Convai Innovations](https://github.com/NandhaKishorM/laya), com interface local, API, CLI e avaliaÃ§Ã£o reproduzÃ­vel. O SDK upstream permanece disponÃ­vel; a aplicaÃ§Ã£o adicional vive em `practical/`.

**[Guia de uso](https://inematds.github.io/laya/guia/)** Â· **[Curso v2, em repositÃ³rio separado](https://inematds.github.io/laya-curso/)** Â· [README upstream preservado](docs/README-UPSTREAM.md)

## ComeÃ§ar

Python 3.10+ recomendado para a aplicaÃ§Ã£o (testado com 3.12). Internet na primeira carga para baixar o checkpoint; CPU funciona, GPU Ã© opcional.

```bash
git clone https://github.com/inematds/laya.git
cd laya
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install -e . -r practical/requirements.txt
python3 -m practical download
python3 -m practical serve
```

Abra **http://127.0.0.1:8765** no navegador da mesma mÃ¡quina. Porta ocupada? Use `serve --port 8875`. Para CUDA compatÃ­vel, use `python3 -m practical --device cuda serve`. O modelo usa a instalaÃ§Ã£o PyTorch da mÃ¡quina; consulte a distribuiÃ§Ã£o oficial do PyTorch para seu hardware. O guia no GitHub Pages nÃ£o hospeda a API nem roda pesos no navegador.

```bash
python3 -m practical triage --message "Fui cobrado duas vezes e quero o reembolso."
python3 -m practical --device cuda evaluate --output avaliacao.json
python3 -m practical --model-path /caminho/do/checkpoint triage --message "Quero contratar o plano empresarial."
```

`--device`, `--threshold` e `--model-path` vÃªm **antes** do subcomando. O caminho local precisa conter `rl_agent_config.json`, `model.safetensors`, tokenizer e configuraÃ§Ã£o do encoder.

## API

```bash
curl http://127.0.0.1:8765/api/health
curl -X POST http://127.0.0.1:8765/api/triage \
  -H 'Content-Type: application/json' \
  -d '{"message":"Fui cobrado duas vezes.","subject":"CobranÃ§a"}'
```

- `answers`: departamento (`choice`), urgÃªncia (`score` 0â€“2), ameaÃ§a explÃ­cita de cancelamento e pedido explÃ­cito de reembolso (`noul`).
- `policy`: destino sugerido, razÃµes de revisÃ£o e `automated_action_executed: false`.
- `runtime`: dispositivo efetivo, tokens e tempo de inferÃªncia. NÃ£o inclui download/carga; a primeira inferÃªncia ainda tem aquecimento.
- `422`: entrada invÃ¡lida ou maior que o espaÃ§o disponÃ­vel em tokens. `503`: modelo indisponÃ­vel. Falhas nÃ£o viram previsÃµes falsas.

## O que foi adaptado

- Checkpoint **multilingual explÃ­cito** para a fila em portuguÃªs; uma instÃ¢ncia residente por processo.
- ProteÃ§Ã£o contra truncamento silencioso: o tokenizer real verifica se o estado cabe em todas as perguntas.
- InferÃªncia serializada no processo; validaÃ§Ã£o das distribuiÃ§Ãµes, nÃºmeros e categorias antes da polÃ­tica.
- Interface com exemplos, estado de carregamento, erros, leitura do JSON e download voluntÃ¡rio do resultado.
- API local, sem execuÃ§Ã£o de aÃ§Ãµes externas nem persistÃªncia automÃ¡tica dos tickets.
- 16 tickets sintÃ©ticos rotulados e relatÃ³rio com acurÃ¡cia, baseline, Brier e ECE.

## EvidÃªncia e limites

A execuÃ§Ã£o local real em NVIDIA GB10 acertou **13/16 (81,25%)** departamentos. Os erros e distribuiÃ§Ãµes estÃ£o em [docs/avaliacao-local.json](docs/avaliacao-local.json). A amostra Ã© pequena, sintÃ©tica e nÃ£o valida produÃ§Ã£o. Brier (soma multiclasses): 0,30835; ECE com top probability e 10 bins: 0,17988.

**Toda saÃ­da exige revisÃ£o humana.** O limiar 0,85 serve para acrescentar um motivo de revisÃ£o; nÃ£o autoriza automaÃ§Ã£o. NÃ£o executamos reembolsos, chamadas de agentes ou mudanÃ§as em contas. `act_probability` do upstream nÃ£o Ã© permissÃ£o operacional.

`choice.confidence` Ã© entropia normalizada invertida, nÃ£o probabilidade de acerto. O multilingual nÃ£o estÃ¡ calibrado para o seu domÃ­nio. O nome `churn_risk` neste esquema significa **ameaÃ§a explÃ­cita no texto**, nÃ£o previsÃ£o longitudinal de churn. Score Ã© experimental e deve ser conferido.

A primeira carga baixa pesos do Hugging Face. O texto dos tickets Ã© processado nesta mÃ¡quina. Self-hosted evita tarifa Laya por chamada, mas tem custo de hardware e operaÃ§Ã£o. O servidor liga somente em loopback e Ã© um laboratÃ³rio local; nÃ£o inclui autenticaÃ§Ã£o, rate limit distribuÃ­do, fila durÃ¡vel ou endpoint pÃºblico.

## VerificaÃ§Ã£o

```bash
python3 -m pip install httpx
python3 -m unittest discover -s tests/practical -v
python3 tests/test_router.py
python3 tests/test_criteria.py
python3 -m practical --device cuda evaluate --output avaliacao.json
```

Os dois testes upstream sÃ£o **scripts**, nÃ£o mÃ³dulos pytest. O teste `test_local_e2e.py` do upstream requer checkpoints adicionais e nÃ£o faz parte dessa suÃ­te mÃ­nima.

## ProveniÃªncia e licenÃ§a

Base upstream: `42626c348753fbb17572a813127df2278a1ec527`. AdaptaÃ§Ã£o INEMA: **v0.4.4**. [AnÃ¡lise das fontes](docs/ANALISE.md) Â· [Changelog](CHANGELOG.md) Â· [LicenÃ§a Apache 2.0](LICENSE).

CÃ³digo e pesos continuam atribuÃ­dos aos autores originais. Materiais integrais de terceiros (vÃ­deo, transcriÃ§Ã£o, artigos) ficam no acervo local de pesquisa; o repositÃ³rio publica sÃ­ntese autoral, referÃªncias e evidÃªncias do laboratÃ³rio. NÃ£o foi executado Jev nem treinamento de um novo modelo nesta entrega.

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe