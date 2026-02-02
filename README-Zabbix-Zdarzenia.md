# 🚨 Zabbix - Zdarzenia (problem.get)

Prosty raport aktywnych problemów z Zabbix wysyłany na e-mail.

## 📋 Opis

Lekki workflow pobierający ostatnie 10 aktywnych problemów z Zabbix i wysyłający raport HTML na e-mail. Idealny jako punkt startowy do nauki integracji n8n z Zabbix.

## 🔄 Schemat działania

```
Manual Trigger
    │
    ▼
Zabbix API (problem.get)
    │
    ▼
Generate HTML (JavaScript)
    │
    ▼
Send Email
```

## ⚙️ Wymagania

- **n8n** (self-hosted lub cloud)
- **Zabbix 5.0+** z włączonym API
- **Serwer SMTP**

## 🔧 Konfiguracja

### 1. Zabbix API Token
Utwórz credentials w n8n:
- **Type:** Bearer Auth
- **Token:** API Token z Zabbix

### 2. SMTP
Skonfiguruj credentials SMTP.

### 3. URL Zabbix
Zaktualizuj w node `Zdarzenia`:
```
http://192.168.1.50/zabbix/api_jsonrpc.php
```

## 📡 Zapytanie API

```json
{
  "jsonrpc": "2.0",
  "method": "problem.get",
  "params": {
    "output": "extend",
    "recent": true,
    "sortfield": ["eventid"],
    "sortorder": "DESC",
    "limit": 10
  },
  "id": 2
}
```

## 📊 Format raportu

```html
<h2>Raport Zabbix – ostatnie problemy</h2>
<ul>
  <li>
    <b>EventID:</b> 12345
    <b>Opis:</b> High CPU usage
    <b>Severity:</b> 4
    <b>Czas:</b> 02.02.2026, 10:30:00
  </li>
</ul>
```

## 🔢 Severity Levels

| Wartość | Poziom |
|---------|--------|
| 0 | Not classified |
| 1 | Information |
| 2 | Warning |
| 3 | Average |
| 4 | High |
| 5 | Disaster |

## 💡 Rozszerzenia

- Dodaj `Schedule Trigger` dla automatyzacji
- Filtruj po severity: `"severities": [4, 5]`
- Dodaj `selectHosts` dla nazw hostów
- Integracja z AI dla podsumowania (patrz: Zabbix-Triggery-24h)

## 🆚 Różnica: problem.get vs event.get

| Metoda | Użycie |
|--------|--------|
| `problem.get` | Tylko aktywne problemy |
| `event.get` | Wszystkie zdarzenia (problemy + recovery) |

## 🏷️ Tagi

`zabbix` `monitoring` `alerting` `email` `automation` `beginner`

## 📝 Dokumentacja

- [Zabbix API - problem.get](https://www.zabbix.com/documentation/current/en/manual/api/reference/problem/get)

## 📄 Licencja

MIT License
