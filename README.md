ï»¿# Laya ARCKHÉ ê¢â¬â triagem prêÂ¡tica em portuguêÂªs

**ê°Å¸â¡Â§ê°Å¸â¡Â· [PortuguêÂªs](README.md) êÂ· ê°Å¸â¡Âºê°Å¸â¡Â¸ [English](README.en.md) êÂ· ê°Å¸â¡Âªê°Å¸â¡Â¸ [EspaêÂ±ol](README.es.md)**

AdaptaêÂ§êÂ£o do [Laya, da Convai Innovations](https://github.com/NandhaKishorM/laya), com interface local, API, CLI e avaliaêÂ§êÂ£o reproduzêÂ­vel. O SDK upstream permanece disponêÂ­vel; a aplicaêÂ§êÂ£o adicional vive em `practical/`.

**[Guia de uso](https://linnoff88-dev.github.io/arckhe-/guia/)** êÂ· **[Curso v2, em repositêÂ³rio separado](https://linnoff88-dev.github.io/arckhe-/)** êÂ· [README upstream preservado](docs/README-UPSTREAM.md)

## ComeêÂ§ar

Python 3.10+ recomendado para a aplicaêÂ§êÂ£o (testado com 3.12). Internet na primeira carga para baixar o checkpoint; CPU funciona, GPU êÂ© opcional.

```bash
git clone https://github.com/linnoff88-dev/laya.git
cd laya
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install -e . -r practical/requirements.txt
python3 -m practical download
python3 -m practical serve
```

Abra **http://127.0.0.1:8765** no navegador da mesma mêÂ¡quina. Porta ocupada? Use `serve --port 8875`. Para CUDA compatêÂ­vel, use `python3 -m practical --device cuda serve`. O modelo usa a instalaêÂ§êÂ£o PyTorch da mêÂ¡quina; consulte a distribuiêÂ§êÂ£o oficial do PyTorch para seu hardware. O guia no GitHub Pages nêÂ£o hospeda a API nem roda pesos no navegador.

```bash
python3 -m practical triage --message "Fui cobrado duas vezes e quero o reembolso."
python3 -m practical --device cuda evaluate --output avaliacao.json
python3 -m practical --model-path /caminho/do/checkpoint triage --message "Quero contratar o plano empresarial."
```

`--device`, `--threshold` e `--model-path` vêÂªm **antes** do subcomando. O caminho local precisa conter `rl_agent_config.json`, `model.safetensors`, tokenizer e configuraêÂ§êÂ£o do encoder.

## API

```bash
curl http://127.0.0.1:8765/api/health
curl -X POST http://127.0.0.1:8765/api/triage \
  -H 'Content-Type: application/json' \
  -d '{"message":"Fui cobrado duas vezes.","subject":"CobranêÂ§a"}'
```

- `answers`: departamento (`choice`), urgêÂªncia (`score` 0ê¢â¬â2), ameaêÂ§a explêÂ­cita de cancelamento e pedido explêÂ­cito de reembolso (`noul`).
- `policy`: destino sugerido, razêÂµes de revisêÂ£o e `automated_action_executed: false`.
- `runtime`: dispositivo efetivo, tokens e tempo de inferêÂªncia. NêÂ£o inclui download/carga; a primeira inferêÂªncia ainda tem aquecimento.
- `422`: entrada invêÂ¡lida ou maior que o espaêÂ§o disponêÂ­vel em tokens. `503`: modelo indisponêÂ­vel. Falhas nêÂ£o viram previsêÂµes falsas.

## O que foi adaptado

- Checkpoint **multilingual explêÂ­cito** para a fila em portuguêÂªs; uma instêÂ¢ncia residente por processo.
- ProteêÂ§êÂ£o contra truncamento silencioso: o tokenizer real verifica se o estado cabe em todas as perguntas.
- InferêÂªncia serializada no processo; validaêÂ§êÂ£o das distribuiêÂ§êÂµes, nêÂºmeros e categorias antes da polêÂ­tica.
- Interface com exemplos, estado de carregamento, erros, leitura do JSON e download voluntêÂ¡rio do resultado.
- API local, sem execuêÂ§êÂ£o de aêÂ§êÂµes externas nem persistêÂªncia automêÂ¡tica dos tickets.
- 16 tickets sintêÂ©ticos rotulados e relatêÂ³rio com acurêÂ¡cia, baseline, Brier e ECE.

## EvidêÂªncia e limites

A execuêÂ§êÂ£o local real em NVIDIA GB10 acertou **13/16 (81,25%)** departamentos. Os erros e distribuiêÂ§êÂµes estêÂ£o em [docs/avaliacao-local.json](docs/avaliacao-local.json). A amostra êÂ© pequena, sintêÂ©tica e nêÂ£o valida produêÂ§êÂ£o. Brier (soma multiclasses): 0,30835; ECE com top probability e 10 bins: 0,17988.

**Toda saêÂ­da exige revisêÂ£o humana.** O limiar 0,85 serve para acrescentar um motivo de revisêÂ£o; nêÂ£o autoriza automaêÂ§êÂ£o. NêÂ£o executamos reembolsos, chamadas de agentes ou mudanêÂ§as em contas. `act_probability` do upstream nêÂ£o êÂ© permissêÂ£o operacional.

`choice.confidence` êÂ© entropia normalizada invertida, nêÂ£o probabilidade de acerto. O multilingual nêÂ£o estêÂ¡ calibrado para o seu domêÂ­nio. O nome `churn_risk` neste esquema significa **ameaêÂ§a explêÂ­cita no texto**, nêÂ£o previsêÂ£o longitudinal de churn. Score êÂ© experimental e deve ser conferido.

A primeira carga baixa pesos do Hugging Face. O texto dos tickets êÂ© processado nesta mêÂ¡quina. Self-hosted evita tarifa Laya por chamada, mas tem custo de hardware e operaêÂ§êÂ£o. O servidor liga somente em loopback e êÂ© um laboratêÂ³rio local; nêÂ£o inclui autenticaêÂ§êÂ£o, rate limit distribuêÂ­do, fila durêÂ¡vel ou endpoint pêÂºblico.

## VerificaêÂ§êÂ£o

```bash
python3 -m pip install httpx
python3 -m unittest discover -s tests/practical -v
python3 tests/test_router.py
python3 tests/test_criteria.py
python3 -m practical --device cuda evaluate --output avaliacao.json
```

Os dois testes upstream sêÂ£o **scripts**, nêÂ£o mêÂ³dulos pytest. O teste `test_local_e2e.py` do upstream requer checkpoints adicionais e nêÂ£o faz parte dessa suêÂ­te mêÂ­nima.

## ProveniêÂªncia e licenêÂ§a

Base upstream: `42626c348753fbb17572a813127df2278a1ec527`. AdaptaêÂ§êÂ£o ARCKHÉ: **v0.4.4**. [AnêÂ¡lise das fontes](docs/ANALISE.md) êÂ· [Changelog](CHANGELOG.md) êÂ· [LicenêÂ§a Apache 2.0](LICENSE).

CêÂ³digo e pesos continuam atribuêÂ­dos aos autores originais. Materiais integrais de terceiros (vêÂ­deo, transcriêÂ§êÂ£o, artigos) ficam no acervo local de pesquisa; o repositêÂ³rio publica sêÂ­ntese autoral, referêÂªncias e evidêÂªncias do laboratêÂ³rio. NêÂ£o foi executado Jev nem treinamento de um novo modelo nesta entrega.

---

Projeto  - ARCKHÉe