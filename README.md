# n8n + Zabbix – raporty mailowe (Problems + Triggery 24h + opcjonalnie AI)

Repo zawiera 2 workflow do n8n, które łączą się z Zabbix API i wysyłają raport na email:
1) **Zabbix - Pobierz Problemy** – pobiera ostatnie problemy (problem.get) i wysyła listę HTML mailem.
2) **Zabbix - Triggery 24h (event.get)** – pobiera eventy triggerów z ostatnich 24h, buduje tabelę HTML, robi krótkie podsumowanie przez OpenAI i wysyła gotowy HTML mailem.

---

## Wymagania
- n8n (self-hosted lub cloud)
- Dostęp do Zabbix API (URL do `api_jsonrpc.php`)
- Konto w Zabbix (username/password) **albo** token (jeśli przerobisz workflow)
- Skonfigurowane **SMTP credentials** w n8n (node: *Send Email*)
- (Dla workflow z AI) **OpenAI API key** w n8n (node: *Podsumowanie AI*)

---

## Import workflow do n8n
1. Wejdź w n8n → **Workflows** → **Import from File**
2. Zaimportuj:
   - `Zabbix - Pobierz Problemy.json`
   - `Zabbix - Triggery 24h (event.get).json`
3. Po imporcie otwórz workflow i skonfiguruj elementy opisane niżej.

> Uwaga: eksport workflow zwykle zawiera tylko ID credentiali (bez sekretów), ale i tak **nie commituj** żadnych haseł/tokenów wpisanych “na sztywno” w node’ach.

---

## Workflow 1: Zabbix - Pobierz Problemy (problem.get)
### Co robi
- **Manual Trigger**
- **Zabbix Login** (HTTP Request) → `user.login`
- **Pobierz Problemy** (HTTP Request) → `problem.get` (ostatnie, `limit: 10`)
- **Code (JS)** → buduje prosty HTML (lista problemów)
- **Send Email** → wysyła HTML

### Co musisz skonfigurować
1) **Zabbix API URL** (w dwóch node’ach HTTP Request):
- Zmień:
  - `http://<adres-IP-Twojego-zabbixa>/zabbix/api_jsonrpc.php`
- Na swój adres (HTTP/HTTPS).

2) **Login do Zabbixa** (node: *Zabbix Login*):
- Podmień w JSON Body:
  - `"username": "Admin"`
  - `"password": "<twoje-haslo-do-zabbixa>"`

3) **Email** (node: *Send Email*):
- Ustaw:
  - `fromEmail`
  - `toEmail`
- Podłącz/wybierz swoje **SMTP credentials** w n8n.

### Jak odpalić automatycznie
Workflow ma **Manual Trigger**. Jeśli chcesz cyklicznie:
- zamień *Manual Trigger* na **Cron** (np. co 15 min / co godzinę) i podłącz w to samo miejsce.

---

## Workflow 2: Zabbix - Triggery 24h (event.get) + AI podsumowanie
### Co robi
- **Manual Trigger**
- **Zabbix Login** → `user.login`
- **Zabbix API - event.get (24h)** → pobiera eventy z `time_from = now - 86400`, sortuje po `clock DESC`, `limit: 1000`
- **Code (JS)** → buduje czytelną tabelę HTML (czas/host/trigger/severity/status/eventid)
- **Podsumowanie AI (OpenAI)** → generuje krótkie podsumowanie + kroki, zwraca HTML jako tekst
- **Send Email** → wysyła HTML z AI

### Co musisz skonfigurować
1) **Zabbix API URL** (node: *Zabbix Login* i *Zabbix API - event.get (24h)*)
- Podmień placeholder jak wyżej.

2) **Login do Zabbixa** (node: *Zabbix Login*)
- Ustaw username/password.

3) **Email** (node: *Send Email*)
- Ustaw `fromEmail`, `toEmail` i SMTP credentials.

4) **OpenAI** (node: *Podsumowanie AI*)
- Ustaw w n8n **OpenAI credentials** (API key).
- Model w workflow jest ustawiony na `gpt-4.1` (możesz zmienić na inny dostępny w Twoim koncie).

---

## Najczęstsze problemy
### 1) Zabbix API zwraca błąd auth / 401
- Upewnij się, że URL prowadzi do poprawnego `api_jsonrpc.php`.
- Sprawdź dane logowania.
- Jeśli używasz Zabbix API tokenów: ten workflow jest pod login/hasło. Da się to uprościć:
  - usuń `user.login`
  - w requestach zostaw `Authorization: Bearer <API_TOKEN>`
  - (wtedy token nie powinien być “wynikiem logowania”, tylko stałym tokenem z Zabbixa)

### 2) Mail przychodzi z `[object Object]`
- W node *Send Email* pole `HTML` musi wskazywać na konkretny string (np. `{{$json.html}}` albo `{{$json.message.content}}`).
- W tym repo jest poprawnie ustawione:
  - Workflow #1: `{{$json.html}}`
  - Workflow #2: `{{$json.message.content}}`

### 3) Zbyt długi raport
- W workflow #2 możesz zmniejszyć `limit` w `event.get` (np. 200) lub filtrować po severity.

---

## Sugestia (pro tip): konfiguracja przez ENV zamiast “na sztywno”
Żeby nie wpisywać adresów/haseł w node:
- ustaw ENV w kontenerze n8n, np. `ZABBIX_URL`, `ZABBIX_USER`, `ZABBIX_PASS`
- w node’ach używaj expression.

Przykład:
- URL: `{{$env.ZABBIX_URL}}`
- Username: `{{$env.ZABBIX_USER}}`
- Password: `{{$env.ZABBIX_PASS}}`

---

## Pliki w repo
- `Zabbix - Pobierz Problemy.json`
- `Zabbix - Triggery 24h (event.get).json`

---

## Licencja
Dodaj jaką chcesz (MIT/Apache-2.0). Jeśli nie wiesz – wybierz MIT.
