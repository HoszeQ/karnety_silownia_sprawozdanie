
Rozdział 4: Normalizacja i analiza
==================================

Baza danych została zaprojektowana w 3NF – wszystkie dane są atomowe i zależą funkcjonalnie od klucza głównego.

Zalety:
- unikanie redundancji
- łatwiejsze aktualizacje

Skrypt pomocniczy (Python)
--------------------------

.. code-block:: python

   import pandas as pd

   df = pd.DataFrame({
       'typ_karnetu': ['miesięczny', 'trzymiesięczny', 'półroczny'],
       'liczba_klientów': [1, 0, 2]
   })

   print(df)

Analiza wydajnościowa
---------------------
Wydajność może być zwiększona poprzez indeksowanie kolumn `klient_id` oraz `ostatnie_wejscie` w tabeli `Wejscie`.
