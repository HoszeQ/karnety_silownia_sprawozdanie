================================
5. Podsumowanie, Wnioski i Repozytoria
================================

Podsumowanie projektu
---------------------
Niniejszy projekt stanowił pełne ćwiczenie inżynierskie, obejmujące cały cykl życia systemu bazodanowego. Rozpoczynając od analizy potrzeb biznesowych fikcyjnej siłowni, poprzez staranne modelowanie danych, aż po fizyczną implementację i analizę aspektów niefunkcjonalnych, projekt ten z powodzeniem przełożył wymagania na działające, bezpieczne i wydajne rozwiązanie. Zastosowanie normalizacji zapewniło integralność danych, podczas gdy świadome planowanie indeksów i polityk bezpieczeństwa przygotowało system do działania w rzeczywistym środowisku.

Wnioski
-------
Realizacja projektu pozwoliła na sformułowanie następujących wniosków:

1.  **Modelowanie jest kluczowe:** Czas poświęcony na staranne stworzenie modelu koncepcyjnego i logicznego procentuje na etapie implementacji i późniejszej konserwacji. Dobrze zaprojektowany schemat jest intuicyjny i łatwy do rozbudowy.
2.  **Normalizacja to kompromis:** Chociaż dążenie do wyższych postaci normalnych jest teoretycznie pożądane, w praktyce należy uwzględniać również wydajność i logikę biznesową. Celowe, udokumentowane odstępstwa (jak przechowywanie ceny w momencie transakcji) są często uzasadnione.
3.  **Wydajność i bezpieczeństwo nie są opcjonalne:** Aspekty te muszą być uwzględniane od samego początku procesu projektowego, a nie dodawane jako "łatki" na końcu. Strategiczne indeksowanie i model bezpieczeństwa oparty na rolach to fundamenty stabilnego systemu.
4.  **Praktyka utrwala teorię:** Projekt ten był nieocenionym doświadczeniem, które pozwoliło na praktyczne zastosowanie i głębsze zrozumienie teoretycznych koncepcji omawianych na zajęciach, takich jak replikacja, partycjonowanie czy zaawansowane strategie backupu.

Możliwe kierunki dalszego rozwoju
---------------------------------
Zaprojektowana baza danych stanowi solidny fundament, który można rozwijać w wielu kierunkach, aby zwiększyć jej wartość biznesową:

* **Moduł rezerwacji zajęć:** Dodanie tabel `Zajecia`, `Instruktorzy` oraz `Rezerwacje` w celu umożliwienia klientom rezerwacji miejsc na zajęciach grupowych.
* **Integracja z systemem płatności:** Połączenie z bramką płatniczą w celu automatyzacji sprzedaży karnetów online.
* **Aplikacja kliencka:** Stworzenie aplikacji mobilnej lub webowej dla klientów, gdzie mogliby sprawdzać ważność swojego karnetu, historię wejść i rezerwować zajęcia.
* **Zaawansowana analityka:** Budowa hurtowni danych i wykorzystanie narzędzi BI (Business Intelligence) do analizy trendów, np. godzin największego obłożenia siłowni, najpopularniejszych typów karnetów czy segmentacji klientów.

Spis repozytoriów
-----------------
1.  **Repozytorium niniejszego sprawozdania:**
    * `https://github.com/HoszeQ/karnety_silownia_sprawozdanie`
2.  **Repozytorium projektu grupowego:**
    * `https://github.com/m-smieja/Kopie_zapasowe_i_odzyskiwanie_danych`