# Renault API (Gigya + Kamereon)

Ruft Fahrzeugdaten (u. a. Akkustand/SoC) über die inoffizielle Renault-API ab. Der Login läuft über Gigya (Identity-Provider), die eigentlichen Fahrzeugdaten über die Kamereon-API.

## Ablauf

```
Login (Gigya)
   │
   ├──> JWT (Gigya)          → id_token, für x-gigya-id_token Header
   │
   └──> AccountInfo (Gigya)  → personId
              │
              └──> Accounts (Kamereon)  → accountId (accountType: MYRENAULT)
                        │
                        └──> Cars (Kamereon)  → VIN
                                  │
                                  └──> SoC (Kamereon)  → batteryLevel
```

`Login` liefert das `login_token`, das für `JWT` und `AccountInfo` gebraucht wird. Das `id_token` aus `JWT` wird für **alle** Kamereon-Requests (`Accounts`, `Cars`, `SoC`) als Header `x-gigya-id_token` benötigt.

## Requests

| Request | Zweck | Setzt Variable |
|---|---|---|
| `Login` | Login mit E-Mail/Passwort | `login_token` |
| `JWT` | Session-Token gegen JWT tauschen | `jwt_token` |
| `AccountInfo` | Person-ID ermitteln | `person_id` |
| `Accounts` | Renault-Account(s) zur Person | `account_id` |
| `Cars` | Fahrzeuge des Accounts (VIN) | `vin` |
| `SoC` | Akkustand abfragen | `soc` |
| `WakeUp` | *(aktuell nicht funktionsfähig)* | – |

Jeder Request setzt in seinem **Post-Response-Script** die benötigte Variable automatisch – Werte müssen nicht mehr manuell kopiert werden. Einfach der Reihe nach "Send" klicken, oder direkt den Diagnose-Request nutzen (s. u.).

## Alles auf einmal: Diagnose-Request

Der Request `0_Diagnose` (Pre-Request-Script) führt die gesamte Kette per `bru.runRequest()` automatisch durch und loggt jeden Schritt in der Konsole. Bei Fehlern zeigt er, in welchem Schritt und mit welchem Fehlercode es gescheitert ist.

**Wichtig:** `bru.runRequest()` liefert die Antwort unter `res.data` (nicht `res.body` wie in normalen Post-Response-Scripts).

## Bekannte Fehlercodes

| Fehler | Bedeutung |
|---|---|
| `403005 Unauthorized user` | `login_token`/`ApiKey` falsch, abgelaufen oder aus falschem Schritt kopiert |
| `err.func.wired.unauthorized` | `x-gigya-id_token` fehlt/abgelaufen/falsch |
| `err.func.wired.not-found` | Falsche URL (Tippfehler, fehlendes Segment) |
| `err.func.not.connected` | `connectedDriver.role` im Account leer → Fahrzeug ist im Account zwar hinterlegt, aber nicht als Fahrer verbunden. Lösung: im Fahrzeug-Multimediasystem unter "Verbindung"/"My Renault Konto" neu koppeln |

## Environment-Variablen (nicht hardcoden!)

```
renault_email
renault_password
gigya_apikey
kamereon_apikey
country          (z. B. DE)
```

Renault/Gigya ändert die API-Keys gelegentlich (siehe [evcc-io/evcc](https://github.com/evcc-io/evcc/tree/master/vehicle/renault) für aktuelle Werte, falls Login mit `Invalid LoginID` fehlschlägt).
