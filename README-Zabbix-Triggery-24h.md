# 📊 Zabbix - Triggery 24h (event.get)

Automatyczny raport z triggerów Zabbix z ostatnich 24 godzin z podsumowaniem AI.

## 📋 Opis

Workflow pobiera zdarzenia triggerów z Zabbix API, generuje tabelę HTML i wykorzystuje OpenAI GPT-4 do stworzenia inteligentnego podsumowania z rekomendacjami działań. Raport wysyłany jest na e-mail.

## 🔄 Schemat działania

```
Manual Trigger / Schedule
    │
    ▼
Zabbix API (event.get)
    │
    ▼
Generate HTML Table (JavaScript)
    │
    ▼
AI Summary (OpenAI GPT-4)
    │
    ▼
Send Email Report
```

## ⚙️ Wymagania

- **n8n** (self-hosted lub cloud)
- **Zabbix 5.0+** z włączonym API
- **OpenAI API Key**
- **Serwer SMTP**

## 🔧 Konfiguracja

### 1. Zabbix API Token
Utwórz credentials w n8n:
- **Type:** Bearer Auth
- **Token:** API Token z Zabbix

Generowanie tokena w Zabbix:
`Administration → Users → API tokens → Create API token`

### 2. OpenAI API
Skonfiguruj credentials OpenAI z kluczem API.

### 3. SMTP
Skonfiguruj credentials SMTP do wysyłki raportów.

### 4. URL Zabbix
Zaktualizuj URL w node `Zdarzenia`:
```
http://192.168.1.50/zabbix/api_jsonrpc.php
```

## 📡 Zapytanie API Zabbix

```json
{
  "jsonrpc": "2.0",
  "method": "event.get",
  "params": {
    "source": 0,
    "object": 0,
    "selectHosts": ["name", "host"],
    "selectRelatedObject": ["description", "priority"],
    "sortfield": ["clock"],
    "sortorder": "DESC",
    "limit": 1000
  },
  "id": 2
}
```

## 📊 Zawartość raportu

### Tabela HTML
| Kolumna | Opis |
|---------|------|
| Czas | Data i godzina zdarzenia |
| Host | Nazwa hosta |
| Trigger | Opis triggera |
| Severity | Poziom (Information → Disaster) |
| Status | PROBLEM / OK |
| EventID | ID zdarzenia |

### Podsumowanie AI
- Treściwe podsumowanie (max 240 znaków)
- Rekomendacje działań priorytetowych
- Analiza wzorców problemów

## 🎨 Mapowanie Severity

```javascript
0: 'Not classified'
1: 'Information'
2: 'Warning'
3: 'Average'
4: 'High'
5: 'Disaster'
```

## ⏰ Automatyzacja

Zamień `Manual Trigger` na `Schedule Trigger`:
```
Cron: 0 8 * * *  // Codziennie o 8:00
```

## 💡 Możliwe rozszerzenia

- Filtrowanie po severity (tylko High/Disaster)
- Integracja ze Slack/Teams
- Generowanie PDF z raportem
- Dashboard w Grafana

## 🏷️ Tagi

`zabbix` `monitoring` `ai` `openai` `reporting` `automation` `devops`

## 📝 Dokumentacja

- [Zabbix API - event.get](https://www.zabbix.com/documentation/current/en/manual/api/reference/event/get)
- [OpenAI API](https://platform.openai.com/docs)

## 📄 Licencja

MIT License
