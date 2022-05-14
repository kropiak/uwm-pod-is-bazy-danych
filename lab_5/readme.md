## **5. Podstawy administracji bazą danych MySQL. Użytkownicy i uprawnienia na poziomie bazy.**

> Zachęcam do prześledzenia zbioru artykułów odnośnie administracji bazą MySQL znajdujących się pod adresem: https://www.mysqltutorial.org/mysql-administration/

### **5.1 Lokalna instalacja serwera MySQL.**

W zależności od systemu operacyjnego (Windows, Linux) proces instalacji może przebiegać inaczej, ale w każdym z tych przypadków należy skonfigurować podstawowe konto administracyjne o nazwie `root`. Dzięki niemu będzie możliwość zalogowania się do podstawowej bazy MySQL i utworzenia nowych baz oraz użytkowników. Hasło do tego konta powinno być silne (duże i małe litery, cyfry i znaki specjalne, długość min. 8 znaków) i zabezpieczone przed dostępem osób trzecich. Jeżeli naszym celem jest utworzenie bazy (baz) na potrzeby innych aplikacji np. serwisu internetowego to w środowisku produkcyjnym należy bezwzględnie unikać wykorzystania tego konta do celów innych niż administracyjne. Oznacza to, że powinniśmy tworzyć bazy i konta użytkowników pod każdą aplikację oddzielnie nadając nowym użytkownikom ograniczone uprawnienia tj. tylko do niezbędnych baz oraz operacji. Więcej szczegółów w dalszej części lekcji.

W przypadku systemu Windows, serwer MySQL będzie zazwyczaj działał jako lokalna usługa, która będzie automatycznie uruchamiana (domyślnie) przy starcie systemu operacyjnego. Jeżeli korzystamy z takie bazy tylko okazjonalnie i nie jest konieczna jej nieprzerwana praca to warto te domyślne ustawienia zmienić, tak zby niepotrzebnie nie używać zasobów procesora i pamięci operacyjnej komputera.


Instalator dla systemu Windows znajdziemy pod adresem: https://dev.mysql.com/downloads/mysql/

Proces instalacji może różnić się w zależności od już zainstalowanych komponentów, więc nie zostanie szczegółowo opisany tutaj. Poniższe linki mogą pomóc w procesie instalacji:
* https://dev.mysql.com/doc/mysql-installation-excerpt/8.0/en/
* https://www.mysqltutorial.org/install-mysql/

### **5.2 Tworzenie nowej bazy danych**

Tworzenie nowej bazy danych to proces wymagający odpowiedniego uprawienia, które domyślnie posiada użytkownik `root`. Polecenie tworzące bazę ma postać:

```sql
CREATE DATABASE [IF NOT EXISTS] nazwa_bazy
[CHARACTER SET charset_name]
[COLLATE collation_name]
```

Jak widać parametry `CHARACTER SET` oraz `COLLATE` są opcjonalne i ich pominięcie spowoduje użycie wartości domyślnych.

Poprzez `CHARACTER SET` możemy ustawić zbiór dopuszczalnych znaków w łańcuchach tekstowych (np. dla języka polskiego, niemieckiego, tureckiego). Jeżeli chcemy aby nasza baza była gotowa na obsługę uniwersalnego zbioru znaków należy wybrać `utf8` lub `ucs2`.

Listę dostępnych zestawów znaków można wyświetlić poleceniem `SHOW CHARACTER SET;`.

> **W systemie MySQL możemy ustawić zestaw znaków na poziomie serwera, bazy, tabeli oraz kolumny !**

Informacje o aktualnych wartościach najłatwiej odszukać poprzez wyświetlenie właściwości danego elementu w MySQL Workbench, ale można to również wykonać odpowiednimi zapytaniami SQL. Więcej w wątku w serwisie StackOverflow: https://stackoverflow.com/questions/1049728/how-do-i-see-what-character-set-a-mysql-database-table-column-is



Więcej można doczytać tutaj:
* https://www.mysqltutorial.org/mysql-character-set/
* https://dev.mysql.com/doc/refman/8.0/en/charset.html

Aby upewnić się, że łącząc się danym klientem do bazy używamy odpowiedniego zestawu znaków możemy po pomyślnym połączeniu wywowałać polecenie `SET NAMES 'utf8';`, które taki zestaw ustawi.

Atrybut `COLLATE` to zbiór reguł porównywania znaków, który chcemy zastosować dla wybranego wcześniej zestawu znaków (CHARACTER SET lub w skrócie CHARSET).

Zgodnie z konwencją nazewniczą nazwy te kończące się na _ci (ang. case insensitive, bez uwzględniania wielkości liter), _cs (ang. case sensitive, z uwzględnieniem wielkości liter) oraz _bin (ang binary, binarnie).

Listę dostępnych reguł można wyświetlić poleceniem:
```sql
# dla nazw zawierających ciąg latin1
SHOW COLLATION LIKE 'latin1%';
```

Więcej:
* https://www.mysqltutorial.org/mysql-collation/
* https://dev.mysql.com/doc/refman/8.0/en/charset.html

### **5.3 Użytkownicy i uprawnienia.**


Nowe konta użytkowników tworzymy zgodnie z postacią polecenia przedstawioną poniżej:

```sql
CREATE USER [IF NOT EXISTS] nazwa_konta
IDENTIFIED BY 'hasło';
```

Nazwa konta składa się z dwóch członów: nazwy użytkownika, znaku @ i nazwy hosta, np.: jkowalski@localhost oznacza użytkownika `jkowalski` z dostępem do serwera z komputera `localhost`. Dzięki temu możemy ograniczać dostęp do serwera MySQL tylko do wybranych hostów. Pominięcie części hosta spowoduje ustawienie wartości `%`, która oznacza dowolny host (niezalecane). 

Jeżeli nazwa uzytkownika lub hosta zawiera znaki specjalne, spacje należy użyć ograniczników np. `'jkowalski'@'localhost'`.

Listę użytkowników możemy wyświetlić poleceniem:
```sql
select user from mysql.user;
```

Usuwanie konta użytkownika odbywa się analogicznie do usuwania tabel, baz:
```sql
DROP USER [IF EXISTS] nazwa_konta [,account_name2]...;
```

Zmiana hasła użytkownika jest również możliwa i służy do tego polecenie `ALTER...`:
```sql
ALTER USER 'user'@'localhost' IDENTIFIED BY 'nowehaslo';
```

Możemy również zablokować konto użytkownika, co uniemożliwi mu korzystanie z możliwości logowania i wykonywania wszystkich dozwolonych wcześniej operacji.

```sql
ALTER USER dolphin@localhost ACCOUNT LOCK;

# można również ustawić ten atrybut w trakcie tworzenia nowego konta
CREATE USER marek@localhost IDENTIFIED BY 'SuperTajneHaslo2000' ACCOUNT LOCK;

# i odblokowanie konta
ALTER USER [IF EXISTS] nazwa_konta ACCOUNT UNLOCK;
```

**Uprawnienia**

Uprawnienia możemy nadawać i zabierać istniejącym kontom w systemie:

```sql
# nadanie uprawnienia
GRANT uprawnienie [,uprawnienie],.. 
ON poziom_uprawnienia 
TO nazwa_konta;

# odebranie uprawnienia
REVOKE uprawnienie [,uprawnienie],.. 
ON poziom_uprawnienia 
FROM nazwa_konta;

# odebranie wszystkich uprawnień
REVOKE 
    ALL [PRIVILEGES], 
    GRANT OPTION 
FROM konto [, konto2];
```

**Przykład:**
```sql
# nadanie uprawnienia SELECT dla jkowalski dla bazy __firma_zti
GRANT SELECT
ON __firma_zti
TO jkowalski@localhost;
```

Wyróżniamy następujące **poziomy uprawnień**:
* globalny
* bazy
* tabeli
* kolumny
* procedury lub funkcji składowanej
* proxy

**Przykłady:**
```sql
# poziom globalny
GRANT SELECT 
ON *.* 
TO jkowalski@localhost;

# poziom bazy
GRANT INSERT 
ON __firma_zti.* 
TO jkowalski@localhost;

# poziom tabeli
GRANT DELETE 
ON __firma_zti.towar 
TO jkowalski@localhsot;

# poziom kolumny
GRANT 
   SELECT (imie ,nazwisko, data_zatrudnienia), 
   UPDATE(nazwisko) 
ON pracownik 
TO jkowalski@localhost;

# poziom 'stored routine' (procedura, funkcja składowana)
GRANT EXECUTE 
ON PROCEDURE sprawdz_cene_sprzedazy 
TO jkowalski@localhost;

# proxy - przeniesienie uprawnień
GRANT PROXY 
ON root 
TO jkowalski@localhost;
```

Pełną listę dostępnych uprawnień oraz szczegółową postać polecenia można znaleźć pod adresem: https://dev.mysql.com/doc/refman/8.0/en/grant.html


Listę uprawnień dla danego konta najszybciej zdobędziemy poleceniem:
```sql
SHOW GRANTS FOR user@host;
```


> Zadania

**Scenariusz**

Stwórz nową bazę o nazwie **test**. Dodaj do niej jakąś tabelę (może kopia istniejącej ?). Utwórz nowego użytkownika o nazwie **czytacz**, który będzie miał uprawnienia tylko do pobierania wierszy z bazy **test** z hosta **localhost**. Użytkownik powinien posiadać również możliwość podłączenia się do tej bazy (zalogowania). Przetestuj poleceniami SQL poprawność uprawnień logując się utworzonym kontem.