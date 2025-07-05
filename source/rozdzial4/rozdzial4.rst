=====================================
4. Normalizacja, Wydajność i Bezpieczeństwo
=====================================

W tym rozdziale przeprowadzono analizę kluczowych niefunkcjonalnych aspektów zaprojektowanej bazy danych.

Analiza normalizacji
--------------------
Normalizacja jest procesem projektowania schematu bazy danych w celu zminimalizowania redundancji danych i wyeliminowania niepożądanych charakterystyk, takich jak anomalie wstawiania, aktualizacji i usuwania. Zaproponowany schemat jest zgodny z **trzecią postacią normalną (3NF)**.

**Zaczynamy od: jednej dużej tabeli (czyli dane nie są jeszcze uporządkowane)**

Wyobraźmy sobie, że wszystko zapisujemy w jednej tabeli `Rejestr_Silowni`.

.. list-table:: Przykład nieuporządkowanej tabeli (`Rejestr_Silowni`)
   :widths: 15 15 20 15 15 25
   :header-rows: 1

   * - Imie_Klienta
     - Nazwisko_Klienta
     - Email_Klienta
     - Typ_Karnetu
     - Cena_Karnetu
     - Daty_Wejsc
   * - Jan
     - Kowalski
     - jan.kowalski@ex.com
     - miesieczny
     - 120.00
     - '2025-07-02, 2025-07-05'
   * - Anna
     - Nowak
     - anna.nowak@ex.com
     - polroczny
     - 500.00
     - '2025-07-03'
   * - Jan
     - Kowalski
     - jan.kowalski@ex.com
     - trzymiesieczny
     - 300.00
     - '2025-08-01, 2025-08-04'

Problemy z taką tabelą:

* **Nie można dodać klienta**, jeśli jeszcze nic nie kupił.
* **Usunięcie jednego wejścia** może przypadkiem usunąć dane o kliencie.
* **Zmiana adresu e-mail** klienta wymaga edytowania kilku wierszy.
* **Dużo powtarzających się danych** — np. Jan Kowalski występuje kilka razy.

**Krok 1: Pierwsza postać normalna (1NF)**

Tabela jest w 1NF, jeśli nie ma list w kolumnach — każda komórka ma jedną wartość.

U nas kolumna `Daty_Wejsc` zawiera listy. Rozbijmy to:

1. Tworzymy tabelę `Klienci_Karnety` — każdy karnet to osobny wiersz.
2. Tworzymy tabelę `Wejscia`, gdzie każde wejście to jeden rekord.

.. list-table:: Tabela `Klienci_Karnety` (po 1NF)
   :widths: 15 20 20 20 15
   :header-rows: 1

   * - KarnetID
     - Imie_Klienta
     - Nazwisko_Klienta
     - Email_Klienta
     - Typ_Karnetu
   * - 1
     - Jan
     - Kowalski
     - jan.kowalski@ex.com
     - miesieczny
   * - 2
     - Anna
     - Nowak
     - anna.nowak@ex.com
     - polroczny
   * - 3
     - Jan
     - Kowalski
     - jan.kowalski@ex.com
     - trzymiesieczny

.. list-table:: Tabela `Wejscia` (po 1NF)
   :widths: 25 25
   :header-rows: 1

   * - KarnetID_FK
     - Data_Wejscia
   * - 1
     - 2025-07-02
   * - 1
     - 2025-07-05
   * - 2
     - 2025-07-03
   * - 3
     - 2025-08-01
   * - 3
     - 2025-08-04

**Problem:** Nadal mamy powtarzające się dane o klientach w `Klienci_Karnety`.

**Krok 2: Druga postać normalna (2NF)**

W 2NF dane powinny zależeć od całego klucza głównego, a nie tylko części.

W naszej tabeli `Klienci_Karnety` dane o kliencie nie zależą od `KarnetID`, tylko od klienta. Trzeba to rozdzielić:

1. Tworzymy tabelę `Klienci` z unikalnym ID.
2. W tabeli `Karnety` trzymamy tylko typ karnetu i odwołanie do klienta.

.. list-table:: Tabela `Klienci` (po 2NF)
   :widths: 20 25 25 30
   :header-rows: 1

   * - KlientID
     - Imie
     - Nazwisko
     - Email
   * - 101
     - Jan
     - Kowalski
     - jan.kowalski@ex.com
   * - 102
     - Anna
     - Nowak
     - anna.nowak@ex.com

.. list-table:: Tabela `Karnety` (po 2NF)
   :widths: 25 25 25
   :header-rows: 1

   * - KarnetID
     - KlientID_FK
     - Typ_Karnetu
   * - 1
     - 101
     - miesieczny
   * - 2
     - 102
     - polroczny
   * - 3
     - 101
     - trzymiesieczny

Tabela `Wejscia` zostaje bez zmian, ale możemy też rozważyć, czy nie lepiej byłoby wiązać ją bezpośrednio z klientem.

**Krok 3: Trzecia postać normalna (3NF)**

Tutaj chodzi o to, żeby dane nie zależały od innych danych niebędących kluczem.

Jeśli w `Karnety` dodamy np. `Cena`, która zależy od `Typ_Karnetu`, to mamy tzw. zależność przechodnią.

Zamiast tego możemy utworzyć tabelę `Cennik` z kolumnami `Typ_Karnetu` i `Cena`.

W naszym projekcie jednak **trzymamy cenę w tabeli `Karnety`**, ponieważ może się ona zmieniać w czasie — to celowe odstępstwo (denormalizacja), żeby zachować historię.

**Podsumowanie**

Zaczęliśmy od nieuporządkowanej tabeli, a skończyliśmy na trzech powiązanych:

* `Klienci`
* `Karnety`
* `Wejscia`

Dzięki temu dane nie powtarzają się, łatwo je edytować i są bezpieczne przed przypadkowymi błędami.

**Wnioski:** Osiągnięty poziom normalizacji zapewnia wysoką integralność danych i elastyczność schematu, jednocześnie zachowując prostotę i zrozumiałość modelu.

Analiza wydajności i indeksowanie
---------------------------------
Wydajność zapytań jest kluczowa dla responsywności systemu. Podstawową techniką optymalizacji jest strategiczne stosowanie indeksów.

**Identyfikacja kandydatów do indeksowania:**
* **Klucze obce:** Kolumny używane jako klucze obce (`Karnety.klient_id`, `Wejscia.klient_id`) są głównymi kandydatami do indeksowania. Indeksy te drastycznie przyspieszają operacje `JOIN` oraz wyszukiwanie rekordów powiązanych z danym klientem. PostgreSQL automatycznie nie tworzy indeksów na kluczach obcych, więc należy je dodać ręcznie.
* **Często filtrowane kolumny:** Kolumna `Karnety.data_waznosci` będzie często używana w klauzuli `WHERE` do sprawdzania aktywnych karnetów. Dodanie na niej indeksu przyspieszy ten krytyczny proces biznesowy.

**Przykładowa implementacja indeksów:**

.. code-block:: sql

   -- Indeks na kluczu obcym w tabeli Karnety
   CREATE INDEX idx_karnety_klient_id ON Karnety(klient_id);

   -- Indeks na kluczu obcym w tabeli Wejscia
   CREATE INDEX idx_wejscia_klient_id ON Wejscia(klient_id);

   -- Indeks wspomagający wyszukiwanie aktywnych karnetów
   CREATE INDEX idx_karnety_data_waznosci ON Karnety(data_waznosci);

**Analiza planu zapytania (`EXPLAIN ANALYZE`):**
Przed dodaniem indeksu `idx_karnety_klient_id`, zapytanie o wszystkie karnety danego klienta skutkowałoby pełnym skanowaniem tabeli (`Seq Scan`). Po jego dodaniu, planer zapytań PostgreSQL wykorzysta znacznie szybszy `Index Scan`, co przy dużej liczbie rekordów może skrócić czas wykonania zapytania z sekund do milisekund.

Zarządzanie bezpieczeństwem
---------------------------
Bezpieczeństwo danych osobowych i operacyjnych jest priorytetem. Zastosowano model bezpieczeństwa oparty na rolach (Role-Based Access Control).

**Definicja ról:**
* **`rola_admin`**: Superużytkownik z pełnymi uprawnieniami do wszystkich tabel (CRUD - Create, Read, Update, Delete). Przeznaczona dla administratorów bazy danych.
* **`rola_recepcja`**: Rola dla pracowników recepcji. Powinna mieć uprawnienia do:
    * `SELECT` na `Klienci`.
    * `INSERT` do `Klienci`.
    * `SELECT`, `INSERT` na `Karnety`.
    * `SELECT`, `INSERT` na `Wejscia`.
    * Brak uprawnień `DELETE` i `UPDATE` na większości danych w celu ochrony przed przypadkowym usunięciem.
* **`rola_analityk`**: Rola tylko do odczytu (`SELECT`) na wszystkich tabelach. Przeznaczona dla analityków biznesowych generujących raporty.

**Przykładowa implementacja ról i uprawnień:**

.. code-block:: sql

   -- Tworzenie ról
   CREATE ROLE rola_recepcja;
   CREATE ROLE rola_analityk;

   -- Nadawanie uprawnień dla recepcji
   GRANT SELECT, INSERT ON Klienci, Karnety, Wejscia TO rola_recepcja;
   GRANT USAGE, SELECT ON SEQUENCE klienci_klient_id_seq, karnety_karnet_id_seq, wejscia_wejscie_id_seq TO rola_recepcja;


   -- Nadawanie uprawnień dla analityka
   GRANT SELECT ON ALL TABLES IN SCHEMA public TO rola_analityk;

   -- Tworzenie użytkowników i przypisywanie im ról
   CREATE USER pracownik_recepcji WITH PASSWORD 'bezpieczne_haslo';
   GRANT rola_recepcja TO pracownik_recepcji;

Skrypty wspomagające
--------------------

.. code-block:: python
   :caption: Skrypty w PostgreSQL

   import psycopg2
   from datetime import date, timedelta

   # ... (konfiguracja połączenia DB_CONFIG) ...

   def generuj_raport_wygasajacych_karnetow(dni_do_konca=7):
       """
       Znajduje klientów, których karnety wygasają
       w ciągu najbliższych 'dni_do_konca' dni.
       """
       # ... (logika połączenia z bazą) ...
       query = """
       SELECT k.imie, k.nazwisko, k.email, kr.data_waznosci
       FROM Klienci k
       JOIN Karnety kr ON k.klient_id = kr.klient_id
       WHERE kr.data_waznosci BETWEEN %s AND %s
       ORDER BY kr.data_waznosci ASC;
       """
       dzis = date.today()
       data_koncowa = dzis + timedelta(days=dni_do_konca)
       cur.execute(query, (dzis, data_koncowa))
       # ... (logika wyświetlania raportu) ...

   def znajdz_najaktywniejszych_klientow(data_od, data_do, limit=5):
    """Wyświetla listę najczęściej wchodzących klientów w danym okresie."""
    print(f"\n--- TOP {limit} najaktywniejszych klientów od {data_od} do {data_do} ---")
    conn = get_connection()
    query = """
    SELECT k.imie, k.nazwisko, COUNT(w.wejscie_id) AS liczba_wejsc
    FROM Wejscia w
    JOIN Klienci k ON w.klient_id = k.klient_id
    WHERE w.data_wejscia::date BETWEEN %s AND %s
    GROUP BY k.klient_id, k.imie, k.nazwisko
    ORDER BY liczba_wejsc DESC
    LIMIT %s;
    """
    with conn.cursor() as cur:
        cur.execute(query, (data_od, data_do, limit))
        for row in cur.fetchall():
            print(f"Klient: {row[0]} {row[1]}, Liczba wejść: {row[2]}")
    conn.close()

    def generuj_raport_sprzedazy(data_od, data_do):
    """Oblicza sumę sprzedaży i liczbę sprzedanych karnetów w danym okresie."""
    print(f"\n--- Raport sprzedaży od {data_od} do {data_do} ---")
    conn = get_connection()
    query = "SELECT COUNT(karnet_id), SUM(cena) FROM Karnety WHERE data_zakupu BETWEEN %s AND %s;"
    with conn.cursor() as cur:
        cur.execute(query, (data_od, data_do))
        result = cur.fetchone()
        print(f"Liczba sprzedanych karnetów: {result[0] or 0}")
        print(f"Łączna kwota sprzedaży: {result[1] or 0.00} PLN")
    conn.close()

.. code-block:: python
   :caption: Skrypty w SQLite

   import psycopg2
   from datetime import date, timedelta

   # --- KONFIGURACJA ---
   DB_FILE = "silownia.db" # Nazwa pliku bazy danych

   def get_connection():
    """Nawiązuje połączenie z bazą danych SQLite."""
    return sqlite3.connect(DB_FILE)

    def znajdz_klientow_z_wygaslym_karnetem_sqlite(dni_od_wyga_do_wyga):
    """Znajduje klientów, których ostatni karnet wygasł w zadanym przedziale dni temu."""
    print(f"\n--- [SQLite] Klienci, których karnet wygasł od {dni_od_wyga_do_wyga[0]} do {dni_od_wyga_do_wyga[1]} dni temu ---")
    conn = get_connection()
    # W SQLite do znalezienia ostatniego karnetu używamy podzapytania z GROUP BY i MAX()
    query = """
    SELECT k.imie, k.nazwisko, k.email, sub.max_data
    FROM Klienci k
    JOIN (
        SELECT klient_id, MAX(data_waznosci) as max_data FROM Karnety GROUP BY klient_id
    ) AS sub ON k.klient_id = sub.klient_id
    WHERE sub.max_data BETWEEN ? AND ?;
    """
    date_to = (date.today() - timedelta(days=dni_od_wyga_do_wyga[0])).isoformat()
    date_from = (date.today() - timedelta(days=dni_od_wyga_do_wyga[1])).isoformat()

    with conn: # Użycie `with conn` automatycznie zarządza transakcjami
        cur = conn.cursor()
        cur.execute(query, (date_from, date_to))
        for row in cur.fetchall():
            print(f"Klient: {row[0]} {row[1]}, Email: {row[2]}, Karnet wygasł: {row[3]}")

   def generuj_raport_sprzedazy_sqlite(data_od, data_do):
    """Oblicza sumę sprzedaży i liczbę sprzedanych karnetów w danym okresie."""
    print(f"\n--- [SQLite] Raport sprzedaży od {data_od} do {data_do} ---")
    conn = get_connection()
    query = "SELECT COUNT(karnet_id), SUM(cena) FROM Karnety WHERE data_zakupu BETWEEN ? AND ?;"
    with conn:
        cur = conn.cursor()
        cur.execute(query, (data_od, data_do))
        result = cur.fetchone()
        print(f"Liczba sprzedanych karnetów: {result[0] or 0}")
        print(f"Łączna kwota sprzedaży: {result[1] or 0.00} PLN")

   def znajdz_najaktywniejszych_klientow_sqlite(data_od, data_do, limit=5):
    """Wyświetla listę najczęściej wchodzących klientów w danym okresie."""
    print(f"\n--- [SQLite] TOP {limit} najaktywniejszych klientów od {data_od} do {data_do} ---")
    conn = get_connection()
    # Używamy funkcji DATE() do wyciągnięcia daty z pełnego timestampa
    query = """
    SELECT k.imie, k.nazwisko, COUNT(w.wejscie_id) AS liczba_wejsc
    FROM Wejscia w
    JOIN Klienci k ON w.klient_id = k.klient_id
    WHERE DATE(w.data_wejscia) BETWEEN ? AND ?
    GROUP BY k.klient_id
    ORDER BY liczba_wejsc DESC
    LIMIT ?;
    """
    with conn:
        cur = conn.cursor()
        cur.execute(query, (data_od, data_do, limit))
        for row in cur.fetchall():
            print(f"Klient: {row[0]} {row[1]}, Liczba wejść: {row[2]}")