# Research ETF — dokumentacja wdrożenia v1.1.x

Prywatna wtyczka ChatGPT: [Research ETF](https://chatgpt.com/plugins/plugins_6ae47eb21904819183360ebac6dfb3c8).

Ten katalog jest **dokumentacją i historią zaleceń**, nie mechanizmem synchronizacji wtyczki z GitHubem. Zmiany samej wtyczki publikuje oddzielnie Plugin Creator.

## Zakres wydania

- Dodany skill `simulate-etf-allocation`: obliczanie liczby całych jednostek ETF z uwzględnieniem budżetu, wag, prowizji, kursów walut i tymczasowej blokady środków.
- Rozszerzona metodologia porównywania historii ETF i benchmarków, w tym zakresu `1R` i niejednakowych dat.
- Rozbudowane wskazówki dla katalogów brokerów o ręczne/automatyczne przewalutowanie i rozdzielenie rzeczywistych kosztów od blokady.
- Analiza nakładania się portfeli i ostrożniejsze porównywanie wyników.
- Testy: blokada 2%, limity wag, niewłaściwy typ CFD, brak kursu FX i dopłaty do istniejącego portfela.

## Granice i aktualność

- Wtyczka działa **wyłącznie odczytowo**: nie wykonuje zleceń ani wymian walutowych.
- Dane cenowe i kursy muszą pochodzić z datowanych źródeł, w idealnym przypadku ceny ASK. Testowe dane z 2026 roku **nie są aktualną ofertą zakupu**.
- Optymalizator przeszukuje ograniczony zbiór kandydatów blisko celów i limitu budżetowego; nie certyfikuje globalnego optimum.
- Przykładowe 2% bufora to parametr historycznego przykładu, nie gwarantowana stawka Bossa.

Instrukcje do dopisania w projekcie ChatGPT: [finanse-project-instructions-addendum.md](finanse-project-instructions-addendum.md).
