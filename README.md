# clash-rules

Mihomo / Clash.Meta rule-sets для триплейн-инфраструктуры.

Используются через `rule-providers` с авто-обновлением раз в сутки. URL'ы тянутся
через jsdelivr CDN — быстрый отклик из РФ + кэш до 12 часов.

## Файлы

| Файл | Behavior | Формат | Назначение |
|---|---|---|---|
| `reject-telemetry.list` | classical | text | Блок-лист телеметрии и трекеров (REJECT) |
| `proxy-chain.list` | classical | text | Сайты/CDN которые принудительно через прокси |
| `category-porn.yaml` | domain | yaml | Категория adult-сайтов (snapshot из MetaCubeX/meta-rules-dat) |

## Обновление category-porn.yaml

Этот файл — снапшот upstream из `MetaCubeX/meta-rules-dat@meta/geo/geosite/category-porn.yaml`.
Для синка с upstream (раз в 1-3 месяца):

```bash
curl -s "https://cdn.jsdelivr.net/gh/MetaCubeX/meta-rules-dat@meta/geo/geosite/category-porn.yaml" \
  -o category-porn.yaml
git add category-porn.yaml && git commit -m "Sync category-porn.yaml with MetaCubeX upstream" && git push
```

## Использование в Mihomo

```yaml
rule-providers:
  telemetry:
    type: http
    behavior: classical
    format: text
    url: "https://cdn.jsdelivr.net/gh/CodeGalaxyExplorer/clash-rules@main/reject-telemetry.list"
    path: ./ruleset/telemetry.list
    interval: 86400
  proxy-chain:
    type: http
    behavior: classical
    format: text
    url: "https://cdn.jsdelivr.net/gh/CodeGalaxyExplorer/clash-rules@main/proxy-chain.list"
    path: ./ruleset/proxy-chain.list
    interval: 86400

rules:
  - RULE-SET,telemetry,REJECT
  - RULE-SET,proxy-chain,Proxy-Chain
  - GEOIP,RU,DIRECT
  - MATCH,Proxy-Chain
```

## Формат

`behavior: classical` — стандартный Clash-синтаксис правил БЕЗ action-части:

```
DOMAIN-SUFFIX,example.com
IP-CIDR,1.2.3.0/24,no-resolve
```

Action (`REJECT`, `Proxy-Chain` и т.п.) задаётся в `rules:` через `RULE-SET,<name>,<action>`.

## Обновление

Mihomo автоматически перекачивает файлы по `interval` (86400 сек = 24ч).
jsdelivr CDN кэширует git-tag `@main` до 12 часов — изменения видны через 12-24 часа.
Для немедленной инвалидации используйте `https://purge.jsdelivr.net/gh/CodeGalaxyExplorer/clash-rules@main/<file>`.
