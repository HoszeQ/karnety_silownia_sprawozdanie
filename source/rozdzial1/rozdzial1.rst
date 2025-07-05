================
1. Wprowadzenie
================

Autor: Szymon Piskorz

Prowadzący: Piotr Czaja  

Niniejsze sprawozdanie stanowi kompleksową dokumentację projektu bazodanowego, którego realizacja przyświecała dwóm głównym celom: dydaktycznemu oraz praktycznemu.

Cel dydaktyczny
---------------
Nadrzędnym celem było pogłębienie wiedzy i zdobycie praktycznych umiejętności z zakresu zaawansowanej administracji systemami baz danych. Proces ten obejmował badanie i aplikację następujących zagadnień:

* **Infrastruktura sprzętowa i konfiguracja:** Analiza wymagań sprzętowych oraz optymalna konfiguracja parametrów serwera bazy danych pod kątem wydajności i stabilności.
* **Kontrola, konserwacja i diagnostyka:** Poznanie narzędzi do monitorowania stanu bazy, diagnozowania wąskich gardeł oraz wdrażanie procedur konserwacyjnych, takich jak aktualizacja statystyk czy reindeksacja.
* **Wydajność, skalowanie i replikacja:** Techniki optymalizacji zapytań, strategie skalowania (pionowego i poziomego) oraz konfiguracja mechanizmów replikacji w celu zapewnienia wysokiej dostępności i rozłożenia obciążenia.
* **Partycjonowanie danych:** Zrozumienie i zastosowanie metod partycjonowania tabel w celu poprawy zarządzania dużymi zbiorami danych i zwiększenia wydajności zapytań.
* **Bezpieczeństwo:** Implementacja polityk bezpieczeństwa, zarządzanie użytkownikami i uprawnieniami, a także ochrona przed nieautoryzowanym dostępem.
* **Kopie zapasowe i odzyskiwanie danych:** Projektowanie i automatyzacja strategii tworzenia kopii zapasowych (pełnych, różnicowych, przyrostowych) oraz procedur odtwarzania danych po awarii.

Cel praktyczny i zakres projektu
---------------------------------
Drugim celem była realizacja kompletnego projektu bazy danych o nazwie "Karnety na siłowni". Projekt ten ilustruje pełen cykl życia produktu bazodanowego, od analizy wymagań, przez modelowanie, aż po wdrożenie i analizę.

**Problem biznesowy:** Współczesne siłownie i kluby fitness obsługują setki, a nawet tysiące klientów. Ręczne zarządzanie karnetami, datami ich ważności oraz rejestracja wejść są nieefektywne, podatne na błędy i uniemożliwiają prowadzenie analiz biznesowych. Projektowana baza danych ma na celu rozwiązanie tych problemów poprzez automatyzację kluczowych procesów.

**Zakres projektu obejmuje:**
* Zarządzanie kartoteką klientów.
* Obsługę sprzedaży i cyklu życia karnetów o różnym okresie ważności.
* Rejestrację i monitorowanie wejść klientów.
* Podstawową analitykę i raportowanie.

**Poza zakresem projektu pozostają:**
* Zarządzanie grafikami zajęć i rezerwacjami.
* Systemy księgowe i fakturowanie.
* Zarządzanie personelem.

Wykorzystane technologie
-------------------------
* **System Bazy Danych:** PostgreSQL 16
* **Język skryptowy:** Python 3.11 (z biblioteką `psycopg2`)
* **System dokumentacji:** Sphinx
* **System kontroli wersji:** Git / GitHub

Struktura sprawozdania
----------------------
Dokument został podzielony na pięć rozdziałów. Po niniejszym wprowadzeniu, rozdział drugi odnosi się do zrealizowanego projektu grupowego. Rozdział trzeci szczegółowo opisuje projekt bazy danych "Karnety na siłowni", prezentując jego modele i procesy. Rozdział czwarty skupia się na analizie normalizacji, wydajności i bezpieczeństwa. Całość zamyka rozdział piąty, zawierający podsumowanie, wnioski oraz listę wykorzystanych repozytoriów.