**Zadanie 1**

Do tabeli `pracownik` dodaj kolumnę `dzial` i ustaw ją jako klucz obcy do tabeli `dzial`. Usuwanie rekordów potomnych powinno być uniemożliwione. Sprawdź czy mechanizm spójności działa. Przypisz pracowników do działów.

**Zadanie 2**

Utwórz tabelę `stanowisko` z kolumnami `id_stanowiska` (klucz główny) oraz `nazwa_stanowiska`. Dodaj rekordy, które będą zawierały wartości z tabeli `pracownik` i kolumny `stanowisko`.

**Zadanie 3**

Zmień definicję kolumny `stanowisko` tak aby była kluczem obcym do tabeli stanowisko. Ustaw pracownikom wartość w tej kolumnie tak, żeby odzwierciedlała wcześniejszą wartość tekstową.

**Zadanie 4**

Zmień definicję klucza obcego w tabeli `pracownik` w kolumnie `dzial` tak, aby przy usunięciu działu z tabeli `dzial` wstawiana była wartość `NULL` w kolumnie `dzial` tabeli `pracownik`.
