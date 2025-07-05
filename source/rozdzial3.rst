
Rozdział 3: Opis bazy danych
=============================

Baza danych dotyczy obsługi klientów siłowni. Główne encje:

- **Klient** (imię, nazwisko, email, telefon)
- **Karnet** (typ, cena, data zakupu, data końca)
- **Wejście** (ostatnie wejście – timestamp)

CREATE TABLE
------------

.. code-block:: sql

   CREATE TABLE Klient (
       id INTEGER PRIMARY KEY AUTOINCREMENT,
       imie TEXT NOT NULL,
       nazwisko TEXT NOT NULL,
       email TEXT UNIQUE NOT NULL,
       telefon TEXT
   );

   CREATE TABLE Karnet (
       id INTEGER PRIMARY KEY AUTOINCREMENT,
       klient_id INTEGER NOT NULL,
       typ TEXT CHECK(typ IN ('miesięczny', 'trzymiesięczny', 'półroczny')),
       cena INTEGER,
       data_zakupu DATE,
       data_konca DATE,
       FOREIGN KEY (klient_id) REFERENCES Klient(id)
   );

   CREATE TABLE Wejscie (
       id INTEGER PRIMARY KEY AUTOINCREMENT,
       klient_id INTEGER NOT NULL,
       ostatnie_wejscie TIMESTAMP,
       FOREIGN KEY (klient_id) REFERENCES Klient(id)
   );

Przykładowe dane
----------------

Klienci:
- Jan Kowalski, jan.kowalski@example.com, 123456789
- Anna Nowak, anna.nowak@example.com, 987654321
- Piotr Wiśniewski, piotr.wisniewski@example.com, 567123890
