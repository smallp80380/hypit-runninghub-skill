---
name: hypit-runninghub
description: Use this skill when installing, configuring, authoring, reviewing, or running the Hypit RunningHub Provider for API-format workflows or RunningHub AI Apps; includes safe CLI preparation, Membership/Coins review, files, outputs, and durable submissions.
---

# Hypit RunningHub Provider

Применяй этот skill вместе с общим `hypit`: общий skill направляет видеоработу, этот описывает конкретно RunningHub Provider. Сначала определи фактический корень установленного Provider (`$PROVIDER_ROOT`): checkout, заданный пользователем, либо корень пакета `@local/provider-runninghub`, найденный через `import.meta.resolve` и его родительский каталог; не угадывай путь. Для точных форматов и текущего поведения читай доступные в этом корне `README.md`, `docs/generic-workflow.md`, `docs/REUSE_RU.md`, `DEVELOPMENT.md` и `package.json`; `PUBLISHING.md` и lockfile нужны для release-работы только в source checkout. Используй текущие файлы и `--help` установленного CLI, а не исторические версии.

## Выбери путь и найди CLI

- **Workflow:** нужен точный service workflow ID и saved workflow либо полный API-format JSON. Inline JSON не отменяет ID. UI-export ComfyUI не считай исполняемым API graph: после импорта он остаётся non-executable до API export или проверенного преобразования.
- **AI App:** используй собственный `webappId` и свежий fetched interface document. `/post/<id>` — Workflow, `/ai-detail/<id>` — App; fetch только читает метаданные, не запускает задачу. Не подменяй один тип другим.
- Один account Endpoint/Module предоставляет обе capability. Не придумывай per-ID Model. Standard Model/LLM Enterprise-Shared не входят в этот RHCoins Workflow/App execution scope; не направляй их через эти capability.
- Для правок графа сохраняй оригинал и используй import/inspect/edit/export. Структурная проверка не устанавливает семантику node, constraints, backend availability или допустимость output index; запроси точный API graph/interface/contracts, если их нет.
- Сначала определи установленный Provider и Hypit Distribution, профиль и точные CredentialRefs. В source checkout вызови CLI по проверенному корню checkout: `node "$PROVIDER_ROOT/bin/rh-workflow.js" --help`. В consumer с уже установленным Provider используй `npx --no-install rh-workflow --help`; не рассчитывай, что bare `rh-workflow` находится в PATH. Если нужно найти пакетный root в consumer, `import.meta.resolve("@local/provider-runninghub")` разрешает экспорт `index.js`; возьми его родительский каталог, не угадывай workstation path. Не устанавливай SDK глобально без явного разрешения пользователя. Не используй dev-only test loader как часть consumer setup.
- CLI-хелперы Provider (`fetch`, импорт, authoring, candidate) работают на Node без Hypit SDK loader. `hypit check/plan/doctor/pricing/build/inspect/get` принадлежат установленному Hypit Runtime/Distribution. Запускай их из нужного workspace с явными путями.
- Примеры команд ниже используют source-checkout prefix `node "$PROVIDER_ROOT/bin/rh-workflow.js"`; в installed consumer замени его на `npx --no-install rh-workflow`. Примеры — только формы команд: до запуска подставь реальные предоставленные user IDs/файлы/параметры, не выдумывай JSON, assets или review facts. Записывай только в явные новые output paths, оригиналы сохраняй.

## Workflow и App: импорт, запрос, author

Workflow:

Перед запуском задай `WORKFLOW_ID` точной ID-строкой, предоставленной пользователем; сохраняй десятичные ID как строки, не преобразуй длинные значения в JavaScript `Number`.
```sh
node "$PROVIDER_ROOT/bin/rh-workflow.js" import-native --input ./workflow-api.json --output ./workflow.document.json
node "$PROVIDER_ROOT/bin/rh-workflow.js" inspect-native --input ./workflow.document.json
node "$PROVIDER_ROOT/bin/rh-workflow.js" request-from-document --input ./workflow.document.json --workflow-id "$WORKFLOW_ID" --values ./values.json --output ./request.json
node "$PROVIDER_ROOT/bin/rh-workflow.js" author-from-request --input ./request.json --id job --files ./assets.json --outputs ./outputs.json --output ./author.svml
```

Для raw API graph вместо `request-from-document` служит `request-from-file`. `values.json` может содержать только `nodeInfoList`, `files`, `bindings`, `review`, `instanceType`. Сохраняй JSON-типы значений; опускай неиспользуемые поля. `instanceType` необязателен и, если указан, только `default|plus|ultra`; не добавляй default или fallback. Файловые поля и binding должны соответствовать реальному графу; универсальных prompt/image/audio ports нет. `inspect-native` должен показать executable API source; если UI-format, остановись до build и получи API export/проверенное преобразование.

App:

```sh
node "$PROVIDER_ROOT/bin/rh-workflow.js" fetch-app --config ./explicit-app-fetch-config.json --output ./app-document.json
node "$PROVIDER_ROOT/bin/rh-workflow.js" inspect-app --input ./app-document.json
node "$PROVIDER_ROOT/bin/rh-workflow.js" request-from-app --input ./app-document.json --values ./app-values.json --output ./app-request.json
node "$PROVIDER_ROOT/bin/rh-workflow.js" author-from-request --input ./app-request.json --id app --files ./app-assets.json --outputs ./app-outputs.json --output ./app.svml
```

Fetch App читает interface по явному CredentialRef, не submit/generation. Request сохраняет interface defaults в порядке, меняя только явные overrides/file bindings. Metadata не раскрывает hidden graph/version и не задаёт универсальные enum, range или requiredness; не угадывай эти свойства. `appDigest` фиксирует интерфейс, но не доказывает неизменность скрытой реализации.

Для Author используй реальные assets с `{key,src,mediaType}`; output selections `{name,index}` указывают фактический порядок query results. `job.result`/`app.result` — composite output; индекс projection добавляй только если нужный индекс подтверждён. Composite `hypit get` выдаёт каталог `value.json` плюс все resources; результат может быть пустым или смешанных типов.

## Candidate и настоящее owner review

`review-candidate` создаёт лишь evidence: `UNREVIEWED_OWNER_MUST_CHECK`, null `reviewedAt` и `expiresAt`. Это не approval и не разрешение на расход. Команда принимает SHA-256 exact credential, но не читает secret и не доказывает Membership:

```sh
node "$PROVIDER_ROOT/bin/rh-workflow.js" review-candidate --input ./request.json --config ./account-config.json --credential-sha256 "$CREDENTIAL_SHA256" --files ./review-files.json --output ./review-candidate.json
```
`CREDENTIAL_SHA256` должен содержать owner-provided hash для точного credential, до запуска; не подставляй буквальный placeholder.

Предпочти hash, переданный владельцем из доверенного credential manager. Canonical helper `credentialSha256` экспортируется из `@local/provider-runninghub/member-binding`; он считает SHA-256 точных UTF-8 bytes secret string без trim/добавленного newline. Если helper недоступен, используй эквивалентный безопасный helper в памяти. Никогда не выводи secret и не раскрывай его в command line, temp file, shell history или логе ради вычисления hash.

Для Workflow с saved ID и без inline graph передай текущий API graph через `--graph`. Для App передай `--app ./app-document.json`, никогда не смешивай review разных kinds. Если настроен App password CredentialRef, его exact hash укажи через `--access-password-sha256`, сам пароль не передавай. Encrypted App требует password при исполнении; unencrypted App пароль не получает, но настроенная ref остаётся частью identity/review.

До review владелец должен подтвердить именно для этой операции:

- actual key принадлежит Membership entitlement; `NORMAL`, положительный баланс, `keyType` label или предыдущий успех этого не доказывают;
- exact Workflow graph/App, все параметры, tier, intended files и отсутствие `consumeMoney`, third-party Money или других расходов вне принятого Coins-only scope;
- пользователь явно утвердил account и spending budget/scope. `submissionEnabled`, candidate, check/plan/doctor и pricing page не являются authorization.

Не создавай approval автоматически. Если владелец явно предоставил и разрешил зафиксировать свои review-факты, можно механически записать именно их в требуемую private record; не выдумывай reviewer, даты, entitlement или подтверждения Money. Candidate не редактируй для имитации approval; Workflow approval не разрешает App и наоборот. Если факт не подтверждён — оставь candidate unreviewed, paid submit запрещён.

File review: Workflow file inventory идёт в порядке файлов самого request/будущих Author `<File>` edges; для каждой позиции фиксируются key, MIME, size и hash фактических bytes. Binding sequence описывает графовые связи, но не задаёт file count/order: один file key может связываться с несколькими fields. Workflow draft может иметь `files: []` и будущие File edges; для review передай реальный inventory будущих Author files, не делая вид, что draft уже содержит их BlobRefs. App candidate, напротив, требует review files ровно в count/order request files. Если App Author draft содержит будущие `<File>` edges, сначала создай отдельный финализированный review request с ordered BlobRefs и настоящими MIME/size; placeholder metadata не годится.

## Credentials, billing и recovery

- Используй только CredentialRefs, явно выбранные пользователем для этого Runtime. CLI metadata-fetch resolver поддерживает только явно настроенный env store. Не ищи `.env`, cookies, shell history, machine auth stores и не копируй существующие secrets; не передавай credential в output hosts. Не публикуй private journals, query evidence или signed URLs.
- `COINS_ONLY` обязателен: никакого NORMAL⇒Membership, monetary fallback, auto-switch key/account/model/GPU или owner approval по умолчанию. В этом Provider исполняются только Workflow/App scope; instanceType не подставляется и не переключается после ошибки.
- Journal identity общая для обоих kinds: account instance + Runtime operation. Изменение запроса, key/password или kind не создаёт новый retry series. Provider допускает до трёх **всего** transient upload/submit попыток с durable accounting; deterministic billing/business errors не ретраятся. Не начинай новую серию, не сбрасывай журнал/lock/state для дополнительных попыток и не дублируй paid submit вручную. Сохраняй state при recovery.
- Durable submit ACK завершает submit: проверяй тот же build/operation через поддерживаемый installed Hypit inspect/status путь, не создавай дубликат и не выдумывай CLI resume command. Unknown/in-flight outcome блокирует новую серию. Положительное или неопределённое Money evidence блокирует продолжение/collection; не удаляй его. Provider не обещает server idempotency или backend availability.

## Проверки, build и результаты

Сначала делай read-only проверки в заданном workspace/profile:

```sh
hypit check ./author.svml --workspace . --json
hypit plan ./run.svrun --runtime ./profile.json --workspace . --json
hypit doctor --runtime ./profile.json --workspace . --json
hypit pricing ./run.svrun --runtime ./profile.json --workspace . --json
```

Сверяй синтаксис с установленным Hypit `--help`; pricing может не давать числовую quote и никогда не разрешает расход. Проверь фактический plan, resources, profile, account, Endpoint/capability и стоимость с пользователем. Только после authentic owner review и явной авторизации расходов на эту операцию можно выполнить:

```sh
hypit build ./run.svrun --runtime ./profile.json --workspace . --follow --json
```

После build пользуйся фактически поддерживаемыми `hypit inspect/get --help` и проверяй тот же build перед collection. Composite `get` сохраняет все resources/safe metadata; index — только projection, не свидетельство MIME. Не теряй empty/mixed/unknown-extension outputs, не заявляй качество или backend-функции без наблюдения.

## Источники, release и расположение skill

Для схем, exact file semantics, Profile, review record, recovery и installed consumer сверяй соответствующие текущие файлы под `$PROVIDER_ROOT`. Для официальных API не выдумывай endpoints/params/enums/response; используй pinned official sources и относящийся к делу контракт, различая Standard Model API от native Workflow/App.

Если пользователь просит release-работу, работай только из source checkout и сначала сверяй текущие `$PROVIDER_ROOT/package.json`, lockfile, `$PROVIDER_ROOT/DEVELOPMENT.md`, `$PROVIDER_ROOT/PUBLISHING.md` и release automation. Версии выводи из текущего source; не меняй старые tags/assets/checksums и не делай старый release Latest. Не публикуй npm, не меняй code/CI, не push/publish/bump version и не выполняй GitHub release operations без явного запроса.

Канонический исходник — корневой `SKILL.md` в [hypit-runninghub-skill](https://github.com/smallp80380/hypit-runninghub-skill). Skill устанавливается отдельно для Codex и OMP в `$HOME/.agents/skills/hypit-runninghub/SKILL.md`; архив Provider его не содержит.
