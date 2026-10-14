# Hypit RunningHub skill

Проектный skill для установки, настройки, безопасного review и запуска Hypit RunningHub Provider: native RunningHub Workflow и AI App на RHCoins. Он не заменяет общий Hypit skill и не устанавливает Provider, Hypit Runtime или SDK.

## Установка для Codex и OMP

Обе системы обнаруживают пользовательские skills в `~/.agents/skills`. Для новой установки убедитесь, что целевой каталог ещё не существует: команда `git clone` завершится ошибкой, а не перезапишет существующие файлы.

```sh
git clone https://github.com/smallp80380/hypit-runninghub-skill.git ~/.agents/skills/hypit-runninghub
```

Для обновления клона используйте fast-forward-only:

```sh
git -C ~/.agents/skills/hypit-runninghub pull --ff-only
```

Если skill уже установлен как обычная копия, не клонируйте поверх неё: обновите её вручную из checkout после проверки изменений, сохраняя существующие файлы. После установки проверьте обнаружение в свежем процессе Codex/OMP.

## Использование

Skill дополняет общий `hypit`; перед работой определите актуальный `$PROVIDER_ROOT` для пользовательского checkout либо установленного `@local/provider-runninghub` и сверяйте там текущие README, workflow/App contract и CLI `--help`. Наличие skill не означает, что Provider или Hypit Runtime установлены.

Для платного исполнения обязательны owner review точной Membership key, Workflow/App, параметров, файлов и Coins-only scope, а также явное разрешение пользователя на расход. Candidate, положительный баланс, `NORMAL`, pricing и `submissionEnabled` не являются authorization. См. подробные безопасные процедуры и recovery в [`SKILL.md`](SKILL.md).
