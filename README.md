# Robot Framework Docker Action

GitHub Action that runs [Robot Framework](https://robotframework.org/) suites using the image  
[`ghcr.io/carlosnizolli/docker-robotframework`](https://github.com/carlosnizolli/docker-robotframework).

Marketplace: [Robot Framework Docker](https://github.com/marketplace/actions/robot-framework-docker)

## Usage

Use a **release tag** that exists on this repository (ex.: `v1.0.2`).  
There is no floating `v1` tag unless you create one (see [Publish](#publish-to-marketplace)).

```yaml
jobs:
  robot:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Robot Framework
        uses: carlosnizolli/robotframework-docker-actions@v1.0.2
        env:
          # Encaminhadas ao container quando definidas (Gemini / robotframework-gemini)
          GEMINI_API_KEY: ${{ secrets.GEMINI_API_KEY }}
          GEMINI_MODEL: gemini-flash-latest
        with:
          tests_path: tests
          reports_path: reports
          options: "--exitonfailure"
          threads: "1"

      - name: Upload reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: robot-reports
          path: reports
```

### Variáveis Gemini (opcional)

Defina no `env` do step. A action só as repassa ao container se estiverem preenchidas:

| Variável | Uso |
|----------|-----|
| `GEMINI_API_KEY` | Chave da API (secret do repositório) |
| `GEMINI_MODEL` | Modelo (ex.: `gemini-flash-latest`) |

Exemplo didático: [robotframework-gemini_exemplos](https://github.com/carlosnizolli/robotframework-gemini_exemplos).

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `image` | no | `ghcr.io/carlosnizolli/docker-robotframework:latest` | Image tag (prefer a fixed version in production) |
| `tests_path` | **yes** | — | Path to tests **relative to the workspace** |
| `reports_path` | no | `reports` | Reports folder relative to the workspace |
| `options` | no | `""` | Extra Robot options (`ROBOT_OPTIONS`) |
| `threads` | no | `1` | `1` → `robot`; `>1` → `pabot` (**desencorajado**) |
| `listener` | no | `""` | Listener args (`ROBOT_LISTENER`), e.g. `--listener MyListener` |
| `pabot_options` | no | `""` | Extra Pabot options when `threads` > 1 (**desencorajado**) |
| `shm_size` | no | `2g` | Docker `--shm-size` (needed for browsers) |

## Parallel execution

### Matrix jobs (recommended)

Para paralelizar testes, prefira **vários jobs** com `strategy.matrix` — cada job roda `robot` com `threads: "1"`, isolado e com relatórios próprios:

```yaml
jobs:
  robot:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        suite:
          - tests/smoke
          - tests/regression
          - tests/api
    steps:
      - uses: actions/checkout@v4

      - name: Run Robot Framework
        uses: carlosnizolli/robotframework-docker-actions@v1.0.2
        with:
          tests_path: ${{ matrix.suite }}
          reports_path: reports/${{ matrix.suite }}
          options: "--exitonfailure"
          threads: "1"

      - name: Upload reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: robot-reports-${{ matrix.suite }}
          path: reports/${{ matrix.suite }}
```

Também funciona com tags via `options`, por exemplo `--include smoke`.

### Pabot (discouraged)

> **Desencorajado.** Prefira matrix jobs ou `threads: "1"` (execução sequencial com `robot`). O Pabot pode gerar instabilidade, conflitos de recursos e relatórios inconsistentes — use apenas se a suíte foi preparada para paralelismo.

```yaml
with:
  tests_path: tests
  threads: "4"
  pabot_options: "--testlevelsplit"
```

## Related

| Project | Role |
|---------|------|
| [docker-robotframework](https://github.com/carlosnizolli/docker-robotframework) | Docker image used by this action |
| [robotframework-gemini](https://github.com/carlosnizolli/robotframework-gemini) | Gemini oracles for RF |
| [robotframework-gemini_exemplos](https://github.com/carlosnizolli/robotframework-gemini_exemplos) | Example suites + CI using this action |
| [RobotToPGListener](https://github.com/carlosnizolli/RobotToPGListener) | Persist results to PostgreSQL |
| [RoboCop](https://github.com/carlosnizolli/RoboCop) | Lint RF suites in CI |

## License

MIT
