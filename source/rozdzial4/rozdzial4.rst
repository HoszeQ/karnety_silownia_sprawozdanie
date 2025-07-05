=====================================
4. Normalizacja, Wydajność i Bezpieczeństwo
=====================================

W tym rozdziale przeprowadzono analizę kluczowych niefunkcjonalnych aspektów zaprojektowanej bazy danych.

Analiza normalizacji
--------------------
Normalizacja jest procesem projektowania schematu bazy danych w celu zminimalizowania redundancji danych i wyeliminowania niepożądanych charakterystyk, takich jak anomalie wstawiania, aktualizacji i usuwania. Zaproponowany schemat jest zgodny z **trzecią postacią normalną (3NF)**.

* **Zgodność z 1NF:** Każda tabela posiada klucz główny, a wszystkie atrybuty zawierają wartości atomowe (niepodzielne).
* **Zgodność z 2NF:** Schemat spełnia drugą postać normalną, ponieważ wszystkie klucze główne są kluczami prostymi (jednokolumnowymi), co automatycznie eliminuje problem częściowych zależności funkcyjnych.
* **Zgodność z 3NF:** W schemacie nie występują zależności przechodnie. Żaden atrybut niekluczowy nie jest funkcyjnie zależny od innego atrybutu niekluczowego. Na przykład, w tabeli `Karnety`, atrybut `cena` jest bezpośrednio zależny od klucza `karnet_id`. Nie ma zależności `karnet_id -> typ_karnetu -> cena`, ponieważ cena jest zapisywana w momencie transakcji, co jest celowym zabiegiem denormalizacyjnym w celu archiwizacji danych historycznych i uodpornienia na zmiany cennika. Gdyby cena była zależna od typu karnetu, należałoby stworzyć osobną tabelę `Cennik`, a w tabeli `Karnety` przechowywać jedynie klucz obcy do niej.

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
Poniższy skrypt w Pythonie został rozszerzony o dodatkową funkcjonalność - wyszukiwanie klientów, którym wkrótce wygasa karnet, co może być użyteczne dla działu marketingu.

.. code-block:: python
   :caption: Zaawansowany skrypt do generowania raportów

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