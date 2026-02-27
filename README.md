# AppSec Automation Toolkit (Python)

Набор скриптов для автоматизации задач Application Security (AppSec): инвентаризация поверхности атаки, базовые web/API проверки, SCA, secrets scanning, проверки контейнеров и Kubernetes манифестов, расчёт риска и генерация отчётности.  
Проект спроектирован так, чтобы его можно было легко использовать локально и в CI/CD.

---

## TL;DR

- **Язык:** Python 3.11+
- **CLI:** `typer` (или `argparse` — на выбор в реализации)
- **Вывод:** JSON/CSV/HTML (PDF опционально)
- **Интеграции:** Jira, SMTP, Webhooks
- **Безопасность:** без `shell=True`, строгая валидация входных данных, rate-limit/таймауты

---

## Что внутри (модули)

### 1) Asset Discovery
- mass health-check URL (status, latency, redirects, TLS errors)
- TLS аудит (протоколы/шифры/сертификат)
- мониторинг истечения сертификатов (threshold)
- проверка security headers (CSP/HSTS/etc)
- CORS анализ (wildcard/origin reflection)
- мониторинг поддоменов и diff между снапшотами

### 2) Web/API Security Testing
- IDOR тестирование (итерация ID, дифф ответов, эвристики ownership)
- rate-limit проверка (burst N запросов / анализ 429/ban)
- API fuzzing (mutation-based; поиск 5xx/аномалий)
- admin/debug endpoint discovery (wordlist)
- SSRF поведение (payloads internal IPs / DNS rebinding patterns — безопасный режим)
- JWT анализ (alg/exp/none, базовые анти-паттерны)
- webhook signature validation (HMAC)

### 3) Dependency & SCA
- парсинг `package.json`, `requirements.txt`, `pom.xml`
- сопоставление с локальной CVE/OSV базой (JSON)
- расчёт CVSS (если есть)
- diff отчётов (new/fixed)

### 4) Secrets Scanner
- regex-based детекторы (AWS keys, tokens, JWT, generic)
- entropy анализ строк
- ignore list/allowlist
- опционально: scan git history
- результат: `file, line, secret_type, confidence`

### 5) Container & K8s Security
- Docker image checks (root user, latest tag, базовый SBOM/пакеты — по реализации)
- K8s YAML checks:
  - privileged, runAsRoot, hostNetwork/hostPID/hostIPC
  - отсутствие resources limits/requests
  - NodePort/LoadBalancer exposure
  - automountServiceAccountToken, allowPrivilegeEscalation, readOnlyRootFilesystem

### 6) Risk Engine
- агрегация CVSS + business criticality
- итоговый risk score (Low/Medium/High/Critical)
- SLA расчёт и overdue

### 7) Reporting
- JSON (machine-readable)
- CSV (таблично)
- HTML executive summary (top risks, диаграммы)
- PDF (опционально)

### 8) Integrations
- Jira: создание задач по findings (severity→priority mapping)
- Webhook alerts
- SMTP alerts
- CI mode: fail pipeline при Critical/по правилам

---

## Структура репозитория (ожидаемая)

```text
appsec_toolkit/
  core/
    models.py
    config.py
    logger.py
    errors.py
  scanners/
    assets/
      healthcheck.py
      tls_audit.py
      cert_expiry.py
      headers.py
      cors.py
      subdomains_diff.py
    webapi/
      idor.py
      ratelimit.py
      fuzz.py
      endpoints.py
      ssrf.py
      jwt.py
      webhook_sig.py
    sca/
      parsers.py
      cve_matcher.py
      diff.py
    secrets/
      regex_rules.py
      entropy.py
      scanner.py
    k8s/
      manifest_audit.py
    containers/
      image_checks.py
  analyzers/
    normalizer.py
    dedupe.py
  reports/
    json_report.py
    csv_report.py
    html_report.py
    pdf_report.py
  integrations/
    jira.py
    smtp.py
    webhook.py
  utils/
    http.py
    fileio.py
    time.py
    rate_limit.py
  cli.py
  config.yaml
  requirements.txt
  Dockerfile
  .github/workflows/ci.yml
  README.md
