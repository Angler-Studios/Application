# Testaufgabe Java Developer - Globales Ticket-System für Minecraft-Netzwerke

## 1. Aufgabenstellung

Entwickle ein serverübergreifendes Support-Ticket-System für Minecraft-Netzwerke.

Spieler können Support-Tickets erstellen, die von Teammitgliedern bearbeitet werden. Das System muss Tickets persistent speichern, Echtzeit-Synchronisation zwischen allen Servern ermöglichen und in einem Multi-Proxy-Netzwerk funktionieren.

## 2. Technische Rahmenbedingungen

### Projektstruktur:

Maven Multi-Module Projekt:

- api
- common
- bukkit
- bungee
### Technologie-Stack
- Java 21
- Gradle
- Paper API 1.21.11
- Velocity API
- MongoDB + Morphia
- Redis + Jedis
- Guava
- Guice
- Lombok

## 3. Funktionale Anforderungen
### 3.1 Ticket-Erstellung

Command:
```
/support
/support <nachricht>
```

#### Verhalten

Ohne Parameter:

- Öffnet ein GUI
- Spieler kann Ticket-Kategorie auswählen

Kategorien:

- PAYMENT
- BUG
- PLAYER_REPORT
- TECHNICAL
- OTHER

Mit Parameter:
```
/support Mein Rang wurde nicht aktiviert
```

→ Ticket wird direkt erstellt.

#### Ticket-Daten

Jedes Ticket enthält:

- Ticket-ID
- Spieler UUID
- Spielername
- Kategorie
- Nachricht
- Server
- Erstellungsdatum
- Status

Status:

```
OPEN
IN_PROGRESS
CLOSED
```

### 3.2 Ticket-Zuweisung

Teammitglieder können Tickets übernehmen.

Command:
```
/ticket claim <id>
```
Nach Übernahme:
- Status → IN_PROGRESS
- Bearbeiter wird gespeichert

Ein Ticket darf nur von einem Moderator gleichzeitig bearbeitet werden.

### 3.3 Echtzeit-Benachrichtigungen
#### Neues Ticket

Redis Channel:
```
tickets:new
```

Alle Teammitglieder mit Permission:
```
ticket.admin
```

erhalten:
```
[TICKET] Neues Ticket von Steve
Kategorie: BUG
Server: Survival
```

### 3.4 Ticket-Kommunikation

Spieler und Moderator können Nachrichten austauschen.

Beispiel:
```
/ticket reply <id> <nachricht>
```
Alle Nachrichten müssen:

- in MongoDB gespeichert werden
- chronologisch abrufbar sein
### 3.5 Multi-Proxy Challenge
#### Problem

Spieler befindet sich auf:

```
Proxy-A
Server: Survival
```

Moderator befindet sich auf:

```
Proxy-C
Server: Lobby
```

Wenn Moderator antwortet:
```
/ticket reply 123 Bitte sende einen Screenshot.
```

muss der Spieler die Nachricht sofort erhalten.

#### Anforderungen

Berücksichtige:

- Proxy-übergreifende Zustellung
- Offline-Spieler
- Duplicate Prevention
- Skalierbarkeit
- Thread-Sicherheit

Zusatzfrage:

Wie würdest du verpasste Ticket-Nachrichten zustellen, wenn ein Spieler offline war?

## 4. Datenmodell
### Ticket
```
@Entity("tickets")
public class TicketModel {

    @Id
    private ObjectId id;

    private UUID playerUuid;
    private String playerName;

    private TicketCategory category;

    private TicketStatus status;

    private UUID assignedTo;
    private String assignedToName;

    private String server;

    private Instant createdAt;
    private Instant updatedAt;
}
```

### TicketMessage
```
@Entity("ticket_messages")
public class TicketMessageModel {

    @Id
    private ObjectId id;

    private ObjectId ticketId;

    private UUID sender;

    private String senderName;

    private String message;

    private Instant createdAt;
}
```

## 5. Redis Messaging
### Channel
```
tickets:new
```

Payload:

```
{
  "ticketId": "123",
  "player": "Steve",
  "category": "BUG",
  "server": "Survival",
  "timestamp": "2026-05-30T12:00:00Z"
}
```

### Channel
```
tickets:update
```

Payload:
```
{
  "ticketId": "123",
  "status": "IN_PROGRESS",
  "moderator": "Admin",
  "timestamp": "2026-05-30T12:10:00Z"
}
```

### Channel
```
tickets:message"
```

Payload:
```
{
  "ticketId": "123",
  "receiverUuid": "uuid",
  "message": "Bitte sende einen Screenshot.",
  "sender": "Admin"
}
```

## 6. Nicht-funktionale Anforderungen
### Performance
- Keine blockierenden Datenbankzugriffe im Main Thread
- Keine eigenen ThreadPools
- Scheduler der Plattform verwenden
- Redis und MongoDB vollständig asynchron
### Sicherheit
- Permission-Checks
- Input-Validierung
- Rate-Limit gegen Ticket-Spam
- Duplicate Detection
### Architektur
- Clean Architecture
- Dependency Injection mit Guice
- Java 21 Features nutzen
- Interfaces für alle Services
## 7. Zusatzaufgaben
1. Entwirf ein Caching-Konzept für offene Tickets.
2. Beschreibe eine Strategie für Redis-Ausfälle.
3. Erkläre, wie du Race Conditions beim gleichzeitigen Claimen eines Tickets verhinderst.
4. Entwirf eine Lösung für Ticket-Metriken:
- Tickets pro Tag
- Durchschnittliche Bearbeitungszeit
- Top 10 Moderatoren nach bearbeiteten Tickets
## 8. Bewertungskriterien
- Architektur & Modularität
- Event-Driven Design
- MongoDB-Modellierung
- Redis-Kommunikation
- Multi-Proxy-Kompetenz
- Thread-Sicherheit
- Code-Qualität
- Skalierbarkeit
- Dokumentation
