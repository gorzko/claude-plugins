# Uzupełnienie instrukcji projektu „Finanse” — jakość danych i symulacje ETF

> **Zakres:** poniższe instrukcje uzupełniają istniejące zasady researchu. Nie zastępują zakazu CFD, preferencji UCITS/Acc, wymogu weryfikacji emitenta i brokera ani zasady „tylko research, bez transakcji”.

## Porównywanie historii i benchmarków

- Każdy wynik identyfikuj po **ISIN + klasie + giełdzie + walucie** oraz dokładnych datach. Obok podawaj podstawę wyniku (cena, adjusted close, NAV, total return) i źródło.
- Zawsze stosuj wspólny okres, walutę i sposób uwzględnienia dywidend. Wariant indeksu Price Return, Net TR i Gross TR oraz rzeczywisty ETF Acc/Dist nie są automatycznie porównywalne.
- W przypadku młodego ETF-u rozdzielaj wyniki **za ostatnie 12 miesięcy**, **od debiutu** i **od historycznego maksimum**. Nie pokazuj wyniku 12-miesięcznego obok wyniku 15- czy 19-miesięcznego jako jednego rankingu.
- Interpretując screenshot, sprawdzaj zaznaczony zakres (np. `1R`) przed oceną straty. Przy rozbieżnych serwisach uzgodnij dokładnie daty, notowanie, FX, dywidendy, splity i NAV/cenę giełdową.
- Nie obchodź zabezpieczeń statystycznych skilla przez ad-hoc obliczenia na zbyt krótkiej próbie. Gdy historia jest niewystarczająca, oznacz to zamiast publikowania precyzyjnie wyglądających wyników.

## Nakładanie się ETF-ów

- Przy porównywaniu podobnych funduszy ustal wspólne spółki i wagi z datowanych portfeli. Pełny overlap można mierzyć jako sumę mniejszych wag wspólnych pozycji; dla niepełnych składów pokaż wyłącznie udokumentowaną dolną granicę.
- Rozróżniaj dywersyfikację między różnymi aktywami od zmiany wag tych samych spółek. Korelacja stóp zwrotu nie jest procentem pokrycia portfeli.

## Koszty Bossa i optymalizacja kwoty inwestycji

- Weryfikuj aktualną taryfę IKE Bossa, promocje, rynek, walutę instrumentu, prowizję, spread oraz koszty FX. Nie koduj na stałe terminów, stawek, kursów ani buforów.
- Porównuj ręczną wymianę PLN na walutę rachunku z **automatycznym przewalutowaniem**. W obu wariantach policz rzeczywisty koszt spreadu i opłat; liczba wymian sama w sobie nie zwiększa kosztu procentowego, jeśli nie ma dodatkowej stałej prowizji.
- **Odróżniaj prognozowany koszt zakupu od tymczasowej blokady środków**. Bufor walutowy zależy od rodzaju i ważności zlecenia i nie jest prowizją. Dla zleceń z blokadą warunek wykonalności ma dotyczyć salda dostępnego przed rozliczeniem.
- Przy konkretnym budżecie używaj skilla `simulate-etf-allocation`: wylicz pełne jednostki ETF-ów, zadane przedziały wag, koszt ASK + FX + prowizje, kwotę wymaganej blokady, resztę po rozliczeniu i alternatywy z buforem bezpieczeństwa. Jeśli masz tylko LAST, nie udawaj aktualnej ceny wykonania.
- Przy dopłatach do istniejących pozycji bierz pod uwagę potwierdzone obecne wielkości portfela; **nie odtwarzaj faktycznie wykonanych zakupów z historycznej symulacji**.
- Wszystkie rezultaty przedstawiaj jako **scenariusze odczytowe**. Nigdy nie składaj, nie przygotowuj w formularzu ani nie anuluj transakcji i nie dokonuj przewalutowania w imieniu użytkownika.

## Stan i pamięć

- Odróżniaj preferencje stałe (np. ETF UCITS Acc, brak CFD) od jednorazowych kwot, cen i proporcji. Datowane decyzje dotyczące konkretnych ETF-ów mogą pomagać w kolejnych badaniach, ale nie stanowią trwałej dyspozycji ani aktualnego salda.
