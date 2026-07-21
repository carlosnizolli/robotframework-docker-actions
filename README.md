# Robot Framework Docker Action

GitHub Action that runs [Robot Framework](https://robotframework.org/) suites using the image  
[`ghcr.io/carlosnizolli/docker-robotframework`](https://github.com/carlosnizolli/docker-robotframework).

## Usage

```yaml
jobs:
  robot:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Robot Framework
        uses: carlosnizolli/robotframework-docker-actions@v1
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

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `image` | no | `ghcr.io/carlosnizolli/docker-robotframework:latest` | Image tag (prefer a fixed version in production) |
| `tests_path` | **yes** | — | Path to tests **relative to the workspace** |
| `reports_path` | no | `reports` | Reports folder relative to the workspace |
| `options` | no | `""` | Extra Robot options (`ROBOT_OPTIONS`) |
| `threads` | no | `1` | `1` → `robot`; `>1` → `pabot` |
| `listener` | no | `""` | Listener args (`ROBOT_LISTENER`), e.g. `--listener MyListener` |
| `pabot_options` | no | `""` | Extra Pabot options when `threads` > 1 |
| `shm_size` | no | `2g` | Docker `--shm-size` (needed for browsers) |

## Parallel (Pabot)

```yaml
with:
  tests_path: tests
  threads: "4"
  pabot_options: "--testlevelsplit"
```

## Publish to Marketplace

1. Push `action.yml` + README to the default branch  
2. Create a release tag (`v1.0.0`) and also move major tag `v1`  
3. In the release UI, check **Publish this Action to the GitHub Marketplace**  
4. Ensure the [GHCR package](https://github.com/carlosnizolli/docker-robotframework/pkgs/container/docker-robotframework) is **public**

```bash
git tag v1.0.0
git push origin v1.0.0
git tag -f v1 v1.0.0
git push origin v1 -f
```

## Related

| Project | Role |
|---------|------|
| [docker-robotframework](https://github.com/carlosnizolli/docker-robotframework) | Docker image used by this action |
| [robotframework-gemini](https://github.com/carlosnizolli/robotframework-gemini) | Gemini oracles for RF |
| [RobotToPGListener](https://github.com/carlosnizolli/RobotToPGListener) | Persist results to PostgreSQL |
| [RoboCop](https://github.com/carlosnizolli/RoboCop) | Lint RF suites in CI |

## License

MIT
