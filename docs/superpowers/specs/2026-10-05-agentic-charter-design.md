# Agentic Charter — спецификация первой версии

Дата: 2026-10-05. Эпик: [#1](https://github.com/sanshan/agentic-charter/issues/1). Задача: [#2](https://github.com/sanshan/agentic-charter/issues/2).

Статус: **согласовано владельцем 2026-10-05** сообщением «Посмотрел давай начнем с этого.»; согласована версия документа в commit `8721c98357d08e66f03bd6c5df6a872b0b3ca7ad`. [План реализации](../plans/2026-10-05-agentic-charter.md) подготовлен отдельно и ожидает review и выбора способа исполнения. Продуктовая реализация ещё не начата. Документ описывает целевое поведение, а не уже работающий продукт.

## 1. Назначение и границы

Один npm-пакет доставляет инженерные стандарты в разные репозитории. Разработчик выбирает профили и получает локально читаемые источники требований, каталог правил, маршрутизацию для агентов, deterministic workflow и инструкцию подключения независимого ревью.

Успех: один владелец общего требования; понятная применимость к конкретному проекту; воспроизводимое обновление; сохранение локальных инструкций; отсутствие ложного заявления о работающем агенте или чистом ревью.

Первая версия охватывает general, Node.js, JavaScript, TypeScript, Nx, NestJS, Next.js, Angular и EDP. Реальные version ranges и технологические правила проходят проверку источников в #7–#10 до включения соответствующего профиля в релиз. Перечень целей не является заявлением о текущей поддержке.

Не входят: веб-сервис, БД каталога, универсальный агентский runtime, автоматическая настройка GitHub/npm аккаунтов, платный LLM API fallback, scheduled curator, automerge каталога, собственный parser свободного текста/реакций Codex. OpenSpec и Superpowers не обязательны для потребителей.

## 2. Журнал согласованных решений

| ID | Решение владельца | Следствие |
| --- | --- | --- |
| D01 | Один npm-пакет с внутренними модулями | Одна версия и согласованный комплект CLI/catalog/templates/adapter |
| D02 | Первый путь ревью — Codex GitHub, как в AccounterBro | Настройка владельцем; ревью после CI; нет API-key Action или credits fallback |
| D03 | Ручной запуск куратора отдельной задачей Codex | Агент готовит draft PR; интеграция PR-review не объявляется task runner |
| D04 | Локальные дополнения и исключения разрешены | Исключение по ID, причине и scope может отключить правило или снизить blocking до advisory |
| D05 | PR не применяет собственное ослабление к себе | Проверка по политике целевой ветки; owner approval; действие после merge |
| D06 | Любой конфликт sync останавливает весь план без записи | Предварительная проверка всех целевых файлов, без частичного разрешения конфликтов |
| D07 | Все изменения центрального каталога принимает владелец | Automerge запрещён; CI, независимое ревью и калибровка остаются обязательными |

Сравнённые варианты:

| Область | Выбранный вариант | Отклонённые варианты и причина |
| --- | --- | --- |
| Поставка | Один автономный npm tarball | Несколько пакетов усложняют совместимость; сетевой bootstrap ухудшает воспроизводимость |
| Sync | Сравнение предыдущего состояния, текущих bytes и новой генерации; остановка на конфликте | Молчаливая перезапись теряет работу; автоматическое слияние текста может изменить смысл |
| Агент | Документированный Codex GitHub review path + capability contract | API-runner требует отдельной оплаты/авторизации; универсальный runtime выходит за границы |
| Куратор | Явная задача владельца → ограниченный PR | Плановый запуск и автоматическое принятие отложены |

## 3. Источники и доказательства

Базовая структура переноса — agentic-workspace-template, ветка bootstrap/reusable-workspace, commit **da3aa6759c206485752c05f6643099188afe7f5a**:
- [Контракт](https://github.com/sanshan/agentic-workspace-template/blob/da3aa6759c206485752c05f6643099188afe7f5a/docs/review/README.md).
- [Валидатор](https://github.com/sanshan/agentic-workspace-template/blob/da3aa6759c206485752c05f6643099188afe7f5a/tools/review/validate-review-catalog.mjs) и его negative tests.
- Каталоги rules/calibration; все canonical sources проверяются в #6.
- Документ интеграции шаблона прямо сообщает, что независимый reviewer для нового репозитория не проверен.

Историческое evidence первого provider path — AccounterBro commit **9e389454ce5976fcad79ead7cca22e1e92586399**:
- [Integration evidence](https://github.com/sanshan/accounterbro/blob/9e389454ce5976fcad79ead7cca22e1e92586399/docs/review/codex-integration.md).
- [Triggering](https://github.com/sanshan/accounterbro/blob/9e389454ce5976fcad79ead7cca22e1e92586399/docs/review/codex-triggering.md).

Эти данные доказывают описанные исторические наблюдения, а не текущую настройку Charter, доступность квоты или сертификацию нового каталога. Не копировать бизнес-ограничения и чужую историческую калибровку как новую.

Для каждого переносимого материала записывать источник, revision, путь, лицензию/право использования и причину адаптации. Приватные архивы не публикуются. PRR-006 остаётся retired; исходные ID сохраняются только при неизменной ответственности, иначе используется явная таблица old → new/retired.

Процесс проектирования: superpowers:brainstorming, [официальный источник](https://github.com/obra/superpowers/blob/8ca22dba9a94f28898bbce59f2537ff4d87c747d/skills/brainstorming/SKILL.md). Архитектурный путь: письменная спецификация → согласование → письменный план → согласование способа исполнения.

## 4. Архитектура пакета

Имя: @sanshan/agentic-charter. Bin: agentic-charter. Репозиторий разработки использует pnpm, TypeScript strict и ESM; один package.json, без Nx и workspace orchestration для единственного пакета. Предлагаемая поддержка CLI: Node.js major 22 и 24, проверяемая CI. Другие major не заявляются автоматически. Версии инструментов фиксируются packageManager и lockfile при #3.

| Модуль / путь | Ответственность | Не делает |
| --- | --- | --- |
| src/catalog | Валидация rules/profiles, разрешение include-графа | Не определяет семантическое нарушение по diff |
| src/config | Проверка local config, exceptions, version compatibility | Не меняет стандарт неявно |
| src/generation | Чистая детерминированная генерация desired outputs | Не пишет файлы и не обращается к сети |
| src/sync | Ownership, preflight, план изменений, запись/восстановление | Не сливает конфликтующий текст |
| src/cli | init/sync/check, диагностика и exit codes | Не запускает агент и не устанавливает зависимости |
| src/review | Схемы result/evidence и описание capabilities провайдера | Не выводит clean из прозы или реакции |
| catalog / standards | JSON правила/профили и Markdown источники требований | Не содержит потребительские бизнес-правила |
| templates | AGENTS routing, deterministic workflow, setup/runbooks | Не заменяет существующий CI |
| fixtures / tests | Контрактные и offline consumer scenarios | Не выдаёт mock за real calibration |

Tarball содержит dist, catalog, standards, schemas, templates и нужную документацию. Секреты, private observations, внутренние transcripts и неиспользуемые fixtures исключены. Установка не имеет скрытого install-hook. Модули внутренние; стабильный публичный контракт первой версии — CLI и versioned schemas.

## 5. Контракты данных

Все JSON схемы имеют schema_version: 1, запрещают неизвестные поля и имеют явные миграции. Повторяемые ID уникальны; пути относительны, ограничены корнем и не проходят через symlink наружу. JSON для digests канонизируется: ключи сортируются, encoding UTF-8, LF, без timestamps; файловые digests считаются по фактическим bytes.

### Rule и источник

Обязательные поля: id, title, category, profiles, scope, applies_when, violation, non_violation, severity, evidence, canonical_source, provenance.

- id: PRR-NNN для унаследованных обязанностей, новые общие — ACR-NNNN; local — LOCAL-<slug>. Retired ID никогда не переиспользуется.
- scope: include/exclude globs и change kinds; applies_when/violation/non_violation — отдельные явные семантические границы.
- severity: advisory или blocking. Новые и семантически изменённые правила advisory до необходимой калибровки.
- canonical_source: один поставляемый Markdown source path с необязательным fragment. Mechanical check проверяет файл; reviewer проверяет раздел и соответствие смысла.
- provenance: origin repository/revision/path, license или permission reference, adaptation note.
- evidence описывает минимальные факты для finding; fixtures и calibration evidence хранятся отдельно по ID.

Semantic digest включает scope, применимость, violation/non_violation, severity, evidence и нормативный source content. Для повышения advisory → blocking trials оценивают proposed blocking definition; evidence относится к semantic digest именно этой кандидатной definition, а не прежней advisory. Изменение нормативного source консервативно инвалидирует evidence; редакционное изменение только title/category не меняет decision boundary. Изменение responsibility получает новый ID.

### Profile и совместимость

Profile: id, title, includes, rule_ids, sources, compatibility, provenance. Граф includes ацикличен, правила дедуплицируются по ID, несовместимые дубликаты — ошибка.

Предлагаемые includes: general без зависимостей; javascript → general; typescript → javascript; node → general; nx → general; nestjs → typescript + node; nextjs → javascript + node (typescript выбирается дополнительно для TS-проекта); angular → typescript; edp → typescript + node. Framework profile не означает, что все его правила относятся ко всем файлам: Node-правила не распространяются на browser-код.

Каждый профиль содержит проверенные ranges соответствующих runtime/packages, режимы и source revisions. Unknown/unsupported target version — явная ошибка выбора профиля, не автоматическое расширение поддержки. Смешанный monorepo использует несколько target mappings; нельзя описывать весь репозиторий одной удобной версией.

Version ranges конкретных технологий — контент задач #7–#10: admission требует primary-source проверки и fixtures соответствующих границ. До этого профиль не выпускается как поддерживаемый. Это не выбор новой архитектуры в каждой задаче.

### Consumer config

agentic-charter.config.json хранит schema_version, profiles, targets, local_sources, local_rules, exceptions, review_provider и workflow package-manager selection. Версии Charter в нём нет.

Targets сопоставляют include/exclude пути с technology versions и режимами (например browser/server). Exact resolved versions из lockfile проверяются против mappings; при невозможности однозначного определения требуется явный target mapping и проверяемый источник версии. Первоначальные lockfile formats: pnpm-lock.yaml и package-lock.json. Другие менеджеры не перезаписываются; первая версия сообщает unsupported lockfile, не создаёт второй lockfile.

Exception содержит rule_id, scope, action (disable или advisory) и непустую reason. Неизвестный ID, увеличение severity через exception, неоднозначно пересекающиеся исключения и попытка изменить процедуру merge — ошибка config. Локальное требование может усиливать конкретную инженерную норму; неоднозначное противоречие разрешается явно, а не порядком чтения файлов.

Precedence: процедура принятия политики → base-approved exceptions → выбранные общие правила + совместимые локальные дополнения. AGENTS сохраняет собственную иерархию для задач разработки, но локальный текст не является скрытым разрешением обходить review policy. Исключения не могут отменить независимость, integrity checks, калибровку или owner approval.

### Generated manifest

.agentic-charter/generated/manifest.json: schema_version, package name/version, selected/resolved profiles, config_digest, catalog_digest, effective_policy_digest, target mappings, file/block ownership, per-output SHA-256 digests и generator schema version.

Manifest — provenance генерации, не второй источник версии и не криптографическая подпись. Check регенерирует expected bytes из установленного пакета и config, проверяет lockfile consistency и сравнивает реальные bytes. Подправить один manifest недостаточно для прохождения check. Если PR меняет сам пакет/lockfile/политику, доверие определяется base policy и review, а не хешем PR-кода.

### Review result и evidence

Внутренний record содержит schema_version, provider, attempt_id, repository, base_sha, head_sha, package version, effective_policy_digest, profiles, CI evidence refs, provider artifact refs, findings и state. Это формат хранения фактов, не обещание получить такой JSON от Codex.

State: unconfigured, pending, blocked, failed, stale, indeterminate, completed. Для failed указывается auth/quota/timeout/cancelled/provider-error/policy-integrity; при неясной причине используется unknown, а не догадка.

completed требует отдельного assessment: clean или findings, origin (provider-structured либо owner-assessed), assessed_by и evidence refs. Первая версия Codex не заявляет provider-structured clean. Owner-assessed record документирует человеческую проверку; он не создаёт успешный машинный review gate и не считается независимым provider artifact.

Finding: POLICY (rule_id + declared severity + location + evidence), CORRECTNESS (severity + location + causal explanation + observable consequence), OPINION (не блокирует). Неопределённость сама по себе не является blocking finding. Policy integrity error останавливает ревью.

## 6. Размещение файлов потребителя и ownership

| Путь | Владелец |
| --- | --- |
| agentic-charter.config.json | Потребитель; init создаёт один раз, sync не переписывает |
| .agentic-charter/local/ | Потребитель: источники, правила, исключения при необходимости через ссылки config |
| .agentic-charter/generated/ | Пакет: выбранные rules/sources, effective policy, manifest, setup instructions |
| AGENTS.md, блок agentic-charter | Пакет владеет только отмеченным блоком маршрутизации |
| .github/workflows/agentic-charter-check.yml | Пакет, только если путь свободен или подтверждён prior ownership |
| Остальные файлы и workflow | Потребитель |

Маршрутизация ведёт к локальному generated bundle, доступному без node_modules. Она не копирует каталог в AGENTS. Nx-managed блок и окружающие bytes сохраняются. Повторяющиеся, отсутствующие при prior ownership или повреждённые managed markers — конфликт.

Generated source references переписываются только внутри принадлежащего пакету bundle. При снятии профиля удаляются исключительно ранее управляемые и неизменённые файлы; чужой файл в том же каталоге сохраняется и диагностируется.

## 7. CLI и безопасное обновление

Владелец устанавливает devDependency выбранной версии своим менеджером; затем запускает локальный bin. Нет скачивания latest при выполнении команд.

- init: принимает --profiles (comma-separated), --non-interactive и --dry-run; интерактивный режим доступен только при TTY. Неинтерактивный запуск без нужных данных возвращает config error. Создаёт отсутствующий config и весь согласованный output одним планом.
- sync: использует существующий config; --dry-run показывает точный plan/diff. Повторный запуск не меняет bytes.
- check: read-only, проверяет config/catalog/compatibility/lockfile/provenance и bytes всех управляемых outputs. Отсутствующие зависимости означают невозможность verification, а не чистое состояние.

Exit codes: 0 — успех/актуально; 1 — drift у check; 2 — config/schema/compatibility; 3 — conflict; 4 — IO/recovery error. Dry-run не пишет и использует те же ошибки preflight; валидный план с изменениями возвращает 0 и явно показывает изменения.

Sync использует три состояния: prior manifest digest, actual current bytes, desired bytes. Unchanged prior content обновляется; уже совпавший desired content оставляется. Изменённый вручную owned content, неизвестное ownership, изменённый удаляемый файл или сторонний файл на desired path дают конфликт. Не выполняется автоматическое принятие чужого файла за baseline.

Сначала проверяется весь план, включая config при init. При любом конфликте целевые файлы не меняются. Затем изменения готовятся во временной области, исходные bytes повторно сверяются перед записью. Запись использует journal/backups; штатная IO-ошибка запускает восстановление. Многофайловая запись не объявляется атомарной при потере питания: незавершённый journal блокирует следующий запуск до восстановления по записанным original bytes. Journal — временный локальный артефакт, не версия стандарта.

Recovery конфликта: показать diff → перенести нужное в local config/sources → восстановить owned content из известного Git baseline → повторить sync. Нет --force для молчаливого уничтожения конфликта.

Обновление: dependency + lockfile + sync outputs одним PR. Downgrade поддерживается только при совместимой schema; иначе восстанавливается целый ранее рабочий commit набора dependency/config/lockfile/outputs. Неизвестная новая schema не переписывается старым CLI.

## 8. Ревью, доверие и workflow

Процесс: self-review → зелёный deterministic CI exact current head → независимое ревью того же head → устранение blocking findings → merge владельцем. Значимый новый commit делает прежнее заключение stale; новая версия policy также требует повторной проверки.

Сгенерированный workflow выполняет только deterministic charter check. Он не заменяет lint/tests/build потребителя. Permissions read-only; checkout не сохраняет credentials; PR-код не получает secrets/write token. Fork PR проверяется без привилегий. Не применять pull_request_target для исполнения head-кода.

Проверка freshness не является review authorization. Изменённые в PR checker/workflow/result не могут сами разрешить этот PR. Для policy-changing PR сравнение authority выполняется по target branch; старый пакет/правила/конфигурация читаются из trusted base отдельно от proposed head. Новые источники/исключения рассматриваются как предмет изменения.

Reviewer проверяет реализацию по действующей base policy. Proposed stronger policy проверяется на целостность/калибровку как отдельное изменение; ослабления не действуют внутри собственного PR. Если новая реализация требует именно такого ослабления, policy PR следует принять отдельно раньше реализации.

В Codex нельзя обещать машинную гарантию, что provider использовал только base instructions. Setup/runbook требует явного review scope и проверки owner; сомнение означает indeterminate, не clean. Структурные CI checks и branch protection уменьшают риск, но не являются доказательством поведения модели.

Все изменения каталога принимаются владельцем. Изменения policy/config/workflow у потребителя также требуют владельца. Исключения не обходят этот процесс.

## 9. Первый provider path: Codex GitHub

Owner setup: подключить GitHub App к нужному репозиторию, настроить связанный Codex account/environment и Code Review; проверить фактические разрешения, триггер и доступность подписочной квоты. Charter не устанавливает App, не создаёт secrets и не включает дополнительные credits. Конкретные текущие UI/условия проверяются по официальной документации при #13; историческая настройка AccounterBro не обещает доступность для любого аккаунта.

Новый PR создаётся draft; после зелёного CI переводится в ready. Если review не началось — однократный @codex review. После исправлений сначала новый зелёный CI, затем свежий запрос, поскольку automatic review-on-push не доказан исходным evidence. Истёкшая квота/ненастроенный provider оставляют review невыполненным.

Исторические capabilities:
- finding review имеет commit_id и inline thread metadata;
- clean наблюдался как top-level comment с коротким SHA и/или реакция;
- clean не давал надёжный Codex-owned exact-head check/status;
- поздний clean-looking ответ не устраняет текущий unresolved blocking thread;
- произвольная задача, summary или eyes reaction не доказывают завершённое ревью.

Поэтому машинный gate clean отсутствует. Владелец проверяет completed review, однозначную связь с полным head SHA и отсутствие актуальных blocking threads; неоднозначная реакция/ответ не достаточны. Полный SHA записывается вместе со ссылками на artifacts, но ручное сопоставление не выдаётся за provider-owned статус.

Доступность модели и точное её имя могут быть скрыты провайдером: recording использует provider-managed/unknown. Пакет не обещает выбор модели, unlimited quota или переносимость чужой калибровки.

## 10. Калибровка

Отдельная процедура, не обычный unit CI. Offline tooling готовит case description, diff, evidence skeleton и проверяет report; реальное ревью вызывается через изолированные disposable PR.

Каждое blocking правило требует минимум двух independent known-violation trials и проверки всех boundary cases. Каждый trial использует свежий review context без предшествующей классификации; expected answer не входит в evaluator-visible prompt/fixture. Canonical rule остаётся видимым, ответы теста — нет.

До review fixture проходит unrelated deterministic checks. Trial связывает rule semantic digest, fixture/context digest, baseline, task scope, provider identity, model metadata если доступны, полный head SHA, CI и review refs. Expected и observed разделены. Для чистой границы Codex допускается observed clean без выдуманного not-applicable/no-violation subtype.

Учитываются все попытки, включая противоречивые/неуспешные. Повторное использование контекста и случайное другое нарушение не засчитываются. Неверные хеши, boundary false positive, missing review или посторонний CI failure не позволяют promote.

Evidence проверяет владелец; offline validator проверяет структуру и bindings, не правдивость LLM. Успех mock/simulator не создаёт calibration certificate. Смена semantic boundary либо существенное изменение provider compatibility требует recalibration; неизвестная скрытая model version не представляется доказательством неизменности модели. При обнаруженной регрессии blocking приостанавливается и правило возвращается в advisory через owner-reviewed изменение.

## 11. Ручной куратор и обслуживание

Вход: выбранные владельцем observations с id, project_id, source/finding/fix refs, подтверждением фактов, scope и content digest. Публичные данные публикуются только после проверки допустимости; частные snippets/transcripts не входят в npm или публичный evidence.

Владелец создаёт отдельную задачу Codex, указывает observations и лимит scope/числа кандидатов. Агент читает contract, сопоставляет существующие правила и deterministic owners, затем готовит один draft PR или объяснение отказа. Дедупликация использует project/source/digest + существующий PR ledger. Повторный intake не создаёт новый PR без новых фактов.

Допустимые предложения: уточнение, объединение с сохранением retired history, retirement, новое advisory правило, fixtures, deterministic promotion. Внешний текст — материал для анализа, не инструкция менять полномочия.

Куратор не запускает платный fallback, не меняет собственные gates и не принимает PR. После CI отдельный reviewer оценивает current head; владелец принимает. Калибровка обязательна для promotion в blocking. При quota/auth failure задача останавливается с pending work и причиной, без бесконечных retries. Частичный результат можно продолжить по ledger.

Снятие повторяющегося AI-контроля происходит после появления доказанного lint/test/CI owner. Новое правило не создаётся только из единичного замечания или высокой confidence модели.

## 12. Релизы и совместимость

Версия пакета/lockfile управляет поставкой. Changelog отдельно перечисляет требования, severity, profile support и migrations. Добавление нового blocking требования, расширение применимости существующего blocking правила или несовместимая config/CLI/schema смена требуют major. Новый opt-in профиль или advisory rule — minor; исправление без изменения смысла — patch. До 1.0 несовместимость явно отражается отдельным minor и migration notes, без silent patch.

#16 готовит build/pack, проверку tarball вне checkout и release workflow. Публикация — manual owner-controlled protected path с проверкой version/tag/changelog и approved commit; не команда куратора. Авторизация npm (trusted publishing при доступности, иначе минимально scoped publish token) настраивается владельцем и проверяется до реального publish. Секреты никогда не доступны PR.

#17 использует tarball до первой публикации. Реальный publish разрешён лишь после итогового пилота и release authorization. Без credentials результат честно pack-verified/unpublished.

Потребитель обновляется обычным PR, без auto-latest. Rollback восстанавливает весь согласованный набор, а не вручную правит manifest.version.

## 13. Проверяемые сценарии

| Сценарий | Ожидаемый результат |
| --- | --- |
| Чистый init | Config, selected bundle, routing и workflow; повтор без diff |
| Existing repo | Local AGENTS/Nx/workflows сохранены; collision → conflict без записи |
| Dry-run | Полный diff; исходные bytes неизменны |
| Profile add/remove | Только соответствующий managed output; неизвестный профиль/цикл → ошибка |
| Mixed versions / runtimes | Проверка target compatibility; browser не получает Node finding |
| Upgrade / compatible downgrade | Package+lock+outputs согласованы, идемпотентны |
| Drift / forged manifest | Check nonzero даже при вручную исправленном manifest digest |
| Любой sync conflict | Ни один целевой файл, включая config init, не изменён |
| IO / interruption | Восстановление или явный pending recovery; успех не заявляется |
| Policy weakening PR | Старая политика действует; owner approval; effect после merge |
| Independent review | Только после CI для того же head; self-review не засчитывается |
| New head / policy digest | Предыдущее заключение stale |
| Missing/auth/quota/error/ambiguous review | Нет clean/merge; явное состояние |
| Поздний clean + unresolved finding | Blocking finding сохраняет запрет merge |
| Calibration | Independent trials + boundary, expected не выдаётся evaluator |
| Curator retry / hostile observation | Нет duplicate PR, privilege escalation или утечки |
| Packed consumer без node_modules при review | Локальный bundle читается; CLI check без install не объявляется выполненным |
| Fork PR / modified checker | Нет secrets/write rights или самоподтверждённого merge |

Обычные tests не требуют LLM, внешней сети или БД. Integration tests CLI используют временные fixture repos и tarball. Live provider trials — отдельные evidence, после owner setup.

## 14. Ветки, bootstrap и покрытие эпика

Пустой репозиторий получает минимальный README commit как техническую базу веток, без кода и заявлений об успешных checks. Epic branch: epic/1-agentic-charter. Задача #2: task/2-specification; следующие task/<issue>-<slug> создаются от актуальной epic branch, PR направляются в неё. Epic → main только после интегральной проверки.

Bootstrap не выдаёт self-review за independent review. Spec/plan review владельца — дизайн-gate, а не clean code review. #3 может быть подготовлена в task branch после согласования #2, но merge кода требует работающего CI и независимого review. Начальный documentation-only PR #2 допускается принять владельцем после согласования spec и plan, без заявления о пройденном ещё не существующем CI/независимом review. Это узкое bootstrap-правило только для первоначальных документов, не для каталога или исполняемого кода. После появления #3 checks bootstrap не применяется. Missing provider блокирует merge кода, но не подготовку артефактов. Owner acceptance спецификации не закрывает #2 до принятия плана.

| Задача | Область и выход | Зависимости |
| --- | --- | --- |
| #2 | Согласованные spec, затем plan и способ исполнения | Нет |
| #3 | Toolchain, repository instructions, deterministic CI | #2 |
| #4 | Schemas, catalog/profile resolver, negative validation | #2, #3 |
| #5 | Lifecycle, base-policy authority, owner-only acceptance | #2, #4 |
| #6 | Provenance inventory и перенос исходного каталога | #4, #5 |
| #7 | Node/JS/TS profiles и проверенные ranges | #4, #5, #6 |
| #8 | Nx/Nest profiles | #4, #5, #6, #7 |
| #9 | Next/Angular profiles | #4, #5, #7 |
| #10 | EDP public contract profile | #4, #5, #6, #7 |
| #11 | init и standalone tarball CLI delivery | #3, #4, #5, #6 |
| #12 | sync/check, drift/conflict/recovery | #4, #11 |
| #13 | Consumer workflow и Codex review setup/evidence contract | #2, #5, #11, #12 |
| #14 | Disposable-PR calibration и evidence tools | #4, #5, #13 |
| #15 | Ручная Codex task куратора, intake и ограниченный PR | #5, #6, #12, #13, #14 |
| #16 | Release tooling, tarball, compatibility/rollback | #3, #4, #11, #12, #13 |
| #17 | Интегральный пилот и consumer PR | #6–#16 |

Допустимый последовательный порядок исполнения: #2 → #3 → #4 → #5 → #6 → #7 → #8 → #9 → #10 → #11 → #12 → #13 → #14 → #15 → #16 → #17. Более ранняя подготовка #16 после её зависимостей допускается; публикация всё равно только после #17.

Эпик и #5, #12–#15 уточняются по D01–D07: нет обещания API runner, машинного Codex clean gate, scheduled curator или automerge. Остальные задачи сохраняют границы. Перед каждой реализацией перечитываются эпик и все подзадачи.

## 15. Критерий завершения проектирования

Владелец рассматривает этот документ и явно согласует либо возвращает замечания. После согласования создаётся отдельный implementation plan через superpowers:writing-plans с файлами, проверками, зависимостями и выбранным способом исполнения. Только после его review начинается продуктовая реализация. Публичные claims о поддержке, калибровке и настройке появляются по фактическим evidence соответствующих задач.
