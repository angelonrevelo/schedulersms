![Flutter](https://img.shields.io/badge/Flutter-package-02569B?logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-3-0175C2?logo=dart&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)
![FlutterFlow](https://img.shields.io/badge/works%20with-FlutterFlow-4B39EF)

# SchedulerSMS

**A Flutter package for scheduling SMS messages to customers — on Android, iOS and the web, with built-in support for the Semaphore SMS gateway in the Philippines.**

Write a message, pick a customer and a time, and SchedulerSMS sends it then. Scheduled messages are stored locally in SQLite, can be switched on or off, and report their status as they go out.

- **Per-customer scheduling** — keep a customer list and attach messages to each one.
- **Works where SMS is possible** — sends directly on Android (in the background via WorkManager); on iOS it opens the native composer; on the web it sends through Semaphore or any backend you plug in.
- **Philippine networks** — Globe, Smart and DITO, plus clients for eSIM (Airalo) and prepaid-load APIs.
- **Built for FlutterFlow** — ready-made custom actions in `example/`.
- **Logging** — structured logs you can export as JSON when something goes wrong.

## Install

Add it to `pubspec.yaml`:

```yaml
dependencies:
  schedulersms:
    git:
      url: https://github.com/angelonrevelo/schedulersms
      ref: main
```

```bash
flutter pub get
```

On Android, add `SEND_SMS`, `READ_PHONE_STATE` and `RECEIVE_BOOT_COMPLETED` permissions to `AndroidManifest.xml`. iOS and web need no extra setup — see [doc/internals.md](doc/internals.md) for the exact snippets.

## Quick start (web, with Semaphore)

```dart
import 'package:schedulersms/schedulersms.dart';

final scheduler = SchedulerSmsWebSemaphore();
await scheduler.initialize(apiKey: semaphoreApiKey); // never hardcode the key

final customer = await scheduler.createCustomer(
  name: 'Juan Dela Cruz',
  phoneNumber: '09171234567',
);

await scheduler.scheduleSms(
  customer: customer,
  message: 'Your appointment is tomorrow at 10 AM.',
  scheduledDate: DateTime.now().add(const Duration(hours: 1)),
);
```

Get a Semaphore API key at [semaphore.co](https://semaphore.co/). To check a live key end to end: `dart run example/semaphore_live_test.dart` (balance) or `dart run example/semaphore_live_test.dart <phone>` (sends a test SMS).

## Configuration

The Flutter package reads no environment variables — API keys are passed in as parameters.

The optional eSIM / prepaid-load backend in `backend_api/` (Node, `npm start` or `npm run dev`) reads these from its `.env` (see `backend_api/.env.example`):

- `PORT` — port the API listens on.
- `AIRALO_API_KEY` — Airalo eSIM API key.
- `LOAD_API_UID`, `LOAD_API_PASSWORD` — prepaid-load provider credentials.
- `LOAD_COMPANY_PREFIX` — company prefix for load transaction IDs.

## How it works

```
your app ──▶ SchedulerSMS ──▶ SQLite (customers, scheduled SMS)
                  │
                  ├─ Android: telephony + WorkManager (sends in background)
                  ├─ iOS:     opens the native SMS composer
                  └─ Web:     Semaphore API, or your own sender callback
```

## Limitations

- **iOS** cannot send SMS programmatically; the user taps send in the composer.
- **Web** cannot send SMS by itself; it needs Semaphore or your own backend.
- Aggressive battery optimisation on some Android phones can delay background sends.

## Links

- [doc/internals.md](doc/internals.md) — the full usage reference (customers, logging, FlutterFlow actions, security)
- [doc/](doc/) — FlutterFlow guides, error analysis, testing notes
- [QUICKSTART.md](QUICKSTART.md), [QUICKSTART_SEMAPHORE.md](QUICKSTART_SEMAPHORE.md), [README_SEMAPHORE.md](README_SEMAPHORE.md)
- [backend_api/README.md](backend_api/README.md) — the eSIM / load backend
- [LICENSE](LICENSE) — MIT
