## **4. Ćwiczenia. Wyzwalacze, procedury składowane i funkcje.**

### **4.1 Instrukcja warunkowa.**

W MySQL możemy również wykorzystać instrukcję warunkową, którą możemy określić wartość zwracaną w zapytaniu w zależności od wystąpienia określonej wartości warunku. Do dyspozycji mamy dwie funkcje: `ifnull` oraz `if`.

**Przykład:**
```sql
# ta funkcja ma z góry określony warunek - NULL i w razie jego wystąpienia w kolumnie (pierwszy argument) zwróci wartość podaną jako drugi argument
SELECT pelna_nazwa, ifnull(nip, 'osoba fizyczna') FROM __firma_zti.klient;

# if jest bardziej uniwersalna i pozwala na określenie wartości zarówno jeżeli warunek to prawda lub fałsz
SELECT pelna_nazwa, if(nip is NULL, 'osoba fizyczna', 'firma') FROM __firma_zti.klient;
# lub
SELECT pelna_nazwa, if(isnull(nip), 'osoba fizyczna', 'firma') FROM __firma_zti.klient;

```

### **4.2 Indeksy.**

Indeksy są strukturami, które powodują uporządkowanie danych na dysku dla danej kolumny (kolumn) co powoduje znaczny wzrost szybkości wyszukiwania danych na kolumnie z indeksem. Biorąc to pod uwagę wydawać by się mogło, że założenie indeksów na wszystkie kolumny jest dobrym rozwiązaniem. Dlaczego w takiem razie nie jest to domyślna opcja ? Powodem jest marnowanie zasobów dyskowych w celu przechowywania informacji dla indeksów kolumn, po których wyszukiwanie nie odbywa się zbyt często oraz narzut czasowy MySQL na określenie, którego indeksu należy użyć jeżeli zapytanie dotyczy kilku kolumn. Jak w większości sytuacji - umiar jest wskazany i analiza historii zapytań lub specyfiki aplikacji korzystającej z bazy może tutaj wskazać odpowiednich kandydatów.

Indeksy możemy utworzyć w momencie tworzenia tabeli danych lub poprzez jej modyfikację.

**Przykład:**
```sql
CREATE TABLE czary_mary (nazwa_czaru VARCHAR(100), moc int(11), ksiega enum('magia ognia', 'magia wody', 'magia ziemi', 'magia powietrza'), index(nazwa_czaru));

# przez modyfikację tabeli

CREATE INDEX nazwa_indeksu ON czary_mary(nazwa_czaru);
```

### **Ćwiczenia 1**

W celu utrwalenia wiedzy z zadań poprzednich przygotuj rowzwiązania poniższych zadań:

**Zadanie 1**  
Wyświetl imiona i nazwiska pracowników oraz daty ich urodzenia dla pracowników, którzy będą mieli urodziny w bieżącym miesiącu.

**Zadanie 2**  
Wyświetl imię i nazwisko pracownika A i imię i nazwisko pracownika B, gdzie pracownik A jest co najmniej 10 lat młodszy od pracownika B (self join). Możesz też wyświetlić różnicę wieku w latach.

**Zadanie 3**  
Który miesiąc w całej historii sprzedaży firmy był najlepszy pod względym przychodu ?

**Zadanie 4**  
Wyświetl pełną nazwę klienta oraz w kolejnej kolumnie 'rabat 5%' jeżeli łaczna sprzedaż dla tego klienta przekroczyła 10000 lub 'brak rabatu' jeżeli wartość jest mniejsza.

**Zadanie 5**  
Jaki % łącznej sumy złożonych zamówień przypada na danego sprzedawcę ? Wyświetl 5 najlepszych sprzedawców.


### **4.3 Wyzwalacze.**

Wyzwalacze są strukturami, które są automatycznie uruchamiane przez bazę danych w momencie, w którym warunek ich uruchomienia został spełniony. Takim warunkiem może być operacja wstawienia nowego rekordu, aktualizacji rekordu czy jego usunięcia (INSERT, UPDATE, DELETE). Operacje zadeklarowane w wyzwalaczu mogą zostać wykonane przed (BEFORE) lub po (AFTER) wystąpieniu zdarzenia.
Jako, że domyślnym znakiem oddzielającym zapytania SQL od siebie jest `';'` a taki znak może wystąpić w 'ciele' wyzwalacza wielokrotnie powinniśmy najpierw zmienić ten domyślny ogranicznik na coś co ma małe szanse pokazać się w zapytaniu, np. `//` lub `&&`, które są spotykane najczęściej. Po zakończeniu bloku kodu wracamy do poprzednich ustawień.

**Przykład:**
```sql
DELIMITER //
CREATE TRIGGER towar_before_insert
BEFORE INSERT ON towar
FOR EACH ROW
BEGIN
  IF NEW.cena_zakupu < 0
  THEN
    SET NEW.cena_zakupu = 0;
  END IF;
END
//
DELIMITER ;
```

Powyższy wyzwalacz o nazwie towar_before_insert zostanie uruchomiony w przed operacją `INSERT` na tabeli towar i jeżeli nowy rekord (`NEW`) i jego wartość pola `cena_zakupu` będzie mniejsza od 0, zostanie podstawiona wartość 0. Dopiero wtedy rekord zostanie zapisany do bazy danych. Ta operacja jest powtarzana dla każdego rekordu poprzed pętlę `FOR EACH ROW`.

Istniejące wyzwalacze, lub kod wyzwalacza możemy wyświetlić poniższymi poleceniami:
```sql
SHOW triggers;
SHOW CREATE trigger nazwa;
```

Przykład nieco bardziej rozbudowanego wyzwalacza:

```sql
DELIMITER $$

CREATE TRIGGER uczestnicy_after_insert
AFTER INSERT ON uczestnicy
FOR EACH ROW
BEGIN
	DECLARE tesciowa varchar(100);
	DECLARE sektor_id integer;
	DECLARE czy_tesciowa bool;
	DECLARE czy_chata_dziadka bool;
	SET tesciowa = 'Tesciowa';
	SET sektor_id = 7;
	
/** sprawdzanie czy tesciowa bierze udział w wyprawie **/
    
IF tesciowa in (
	SELECT nazwa from kreatura WHERE idKreatury in 
	( SELECT id_uczestnika from uczestnicy where id_wyprawy=NEW.id_wyprawy)) 
	
THEN 
    
	SET czy_tesciowa = true;
	END IF;
    
IF sektor_id in (
	SELECT sektor FROM etapy_wyprawy WHERE idWyprawy=NEW.id_wyprawy
	) 
THEN 
	SET czy_chata_dziadka = true;
END IF;
    
IF czy_tesciowa AND czy_chata_dziadka
    
THEN  
	INSERT INTO system_alarmowy VALUES(default,'Tesciowa nadchodzi',default);
END IF;

END
$$

DELIMITER ;
```

### **4.3 Procedury i funkcje.**

Procedury i funkcje są strukturami, które podobnie jak wyzwalacze są przechowywane w konkretnej bazie danych. Tworzy się je wtedy kiedy istnieje potrzeba zebrania kilku operacji w jednym miejscu i wywoływania ich wielokrotnie dla różnych danych wejściowych.

**Przykład:**
```sql
DELIMITER $$
CREATE PROCEDURE premia(IN id int)
BEGIN
Update pracownik set pensja = 1.2 * pensja where id_pracownika = id;
END
$$
DELIMITER ;
```

Następnie procedurę wywołujemy poprzez słowo `CALL`.

```sql
CALL premia(10)
```

W powyższym przypadku procedura posiada tylko parametr wejściowy, ale procedura może również zwracać wartość poprzez określenie parametru wyjściowego - `OUT`.

```sql
DELIMITER $$
CREATE PROCEDURE przedstaw_sie(IN id int, OUT powitanie VARCHAR(255))
BEGIN
SELECT concat('Nazywam się ', imie, ' ', nazwisko, '.') INTO powitanie from pracownik where id_pracownika=id;
END
$$
DELIMITER ;

CALL przedstaw_sie(10, @p);
select @p;
```

W powyższej procedurze zadeklarowany jest zarówno parametr wejściowy jak i wyjściowy. Następnie w zapytaniu select wartość zwracana zapisywana jest do zmiennej. Następnie w wywołaniu procedury podawana jest nazwa lokalnej zmiennej, do której wartość zwracana przez procedurę zostanie zapisana. I ostatecznie poprzez SELECT wartość zmiennej jest wyświetlana.

Procedura może również być bezparametryczna.

**Przykład:**
```sql
DELIMITER $$
CREATE PROCEDURE usun_nieaktywnych()
BEGIN
DELETE FROM uzytkownicy WHERE datediff(now(), data_ostatniego_logowania) > 365;
END
$$
DELIMITER ;
```

Przykłady i więcej informacji o procedurach mozna znaleźć pod adresami:
* [W3resource MySQL Procedures](https://www.w3resource.com/mysql/mysql-procedure.php)
* [PG GDA Procedury](http://www.mif.pg.gda.pl/homepages/mate/bazy_danych/procedury.html)


Funkcje to nic innego jak struktury, z których korzystaliśmy wielokrotnie używając funkcji wbudowanych. Jednak teraz możemy zdefiniować je na własne potrzeby w danej bazie. Są podobne do procedur w konstrukcji kodu, ale sposób ich wywołania jest inny.

**Przykład:**
```sql
DELIMITER //
CREATE FUNCTION count_pracownicy()
    RETURNS INTEGER
BEGIN
    DECLARE ile INT;
    SELECT COUNT(*) INTO @ile FROM pracownik;
    RETURN @ile;
END //

select count_pracownicy();
```

Warto wspomnieć o innych różnicach między funkcjami a procedurami aby rozwiać wątpliwości, że zawsze można ich używać zamiennie (w sensie ich samodzielnego tworzenia). Otóż funkcje mają kilka istotnych ograniczeń względem procedur i nie możemy korzystać z transakcji (przynajmniej nie w wersji MySQL, z któryj korzystamy na zajęciach). Transakcja to operacja (lub zbiór operacji), które dzięki mechanizmom bazy danych pozwala na ich atomowe wykonanie i przywrócenie zmian, jeżeli jedna z operacji się nie powiedzie. Transakcje mają dużo więcej cech, ale jest to temat na kolejne laboratoria.

W funkcjach nie są dozwolone poniższe polecenia SQL:
```sql
ALTER 'CACHE INDEX' CALL COMMIT CREATE DELETE
DROP 'FLUSH PRIVILEGES' GRANT INSERT KILL
LOCK OPTIMIZE REPAIR REPLACE REVOKE
ROLLBACK SAVEPOINT 'SELECT FROM table'
'SET zmienna_systemowa' 'SET TRANSACTION'
SHOW 'START TRANSACTION' TRUNCATE UPDATE
```




Oficjalna dokumentacja odnośnie tworzenia własnych funkcji: [MySQL Functions](https://dev.mysql.com/doc/refman/5.7/en/create-function-udf.html) i [User Defined Functions](https://dev.mysql.com/doc/refman/5.7/en/adding-udf.html)

Wykład Doktora Pawła Drozdy nt. wyzwalaczy, procedur i funkcji - http://wmii.uwm.edu.pl/~pdrozda/pliki/wyk8.ppt



### **Ćwiczenia 2**

**Zadanie 1**  
Utwórz kopię lokalną bazy firma_zti.

**Zadanie 2**  
Napisz wyzwalacza, który w momencie dodawania nowego rekordu do tabeli pracownik sprawdzi czy pensja nie jest mniejsza od najniższej krajowej (wstaw dowolną wartość). Jeżeli tak to wstawi wartość najniższej krajowej.

**Zadanie 3**  
Stwórz nową tabelę o nazwie `zamowienia_usuniete`, która będzie zawierała kolumny id_pozycji, numer_zamowienia, data_zamowienia, imie_pracownika, nazwisko_pracownika, pelna_nazwa_klienta, nazwa_towaru, ilosc, nazwa_jm, cena (sprzedaży). Typy kolumn dopasuj tak, żeby odpowiadały kolumnom z bazy firma_zti.

**Zadanie 4**  
Napisz wyzwalacz, który w momencie próby usunięcia rekordu z tabeli zamowienie wstawi rekordy do tabeli `zamowienie_usuniete`. Przetestuj jego działanie.

**Zadanie 5**  
Napisz procedurę o nazwie podnies_ceny, która podniesie ceny wszystkich towarów danej kategorii i podany procent. Mamy więc dwa parametry wejściowe: id_kategorii oraz wartość podwyżki.

**Zadanie 6**  
Napisz funkcję o nazwie inicjaly(), która przyjmuje id pracownika jako parametr wejściowy a zwraca inicjały danego pracownika w postaci np. A.Z. dla Adam Zaręba. Wyświetl zapytaniem select i wykorzystując tę funkcję wszystkich pracowników urodzonych w maju.
