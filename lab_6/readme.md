# Porównanie różnic między bazami SQL i NoSQL na przykładzie bazy MongoDB

## 1. SQL vs. NoSQL

Bazy SQL-owe przechowują dane w uporządkowanym formacie tabelarycznym, gdzie struktura danych jest z góry znana. Może być oczywiście modyfikowana, ale nowe dane trafiające do tabeli muszą się na nią odpowiednio mapować. Sama nazwa SQL (ang. Structured Query Language) już to sugeruje.  
Wraz z rosnącą popularnością oraz liczbą aplikacji www, a przede wszystkim wraz ze zmianą sposobu przesyłania i przechowywania danych pomiędzy klientem a serwerem (najczęściej format JSON), zmieniły się również potrzeby w zakresie dostępu i przechowywania nieustrukturyzowanych danych np. dokumentów, wartości w postaci klucz-wartość, tekstu. Taki rodzaj danych i sposób ich przepływu wpisuje się również w obszar Big Data.

## 2. MySQL vs. MongoDB

**Przechowywanie danych**

W MySQL dane przechowywane są w tabelach, które możemy łączyć relacjami z innymi tabelami za pomocą kluczy obcych i głównych. Stąd też i nazwa - relacyjna baza danych. W MongoDB dane organizowane są w kolekcje, w których rezydują dane i nie ma mechanizmu, który umożliwia stworzenie relacji między tymi danymi. Mówimy tu więc o bazach nierelacyjnych.

**Główne założenia**

Architektura baz SQL-owych oparta jest o założenia wynikające z akronimu ACID. W języku angielskim oznacza to Atomicity, Consistency, Isolation, and Durability czyli niepodzielność (atomowość), spójność, izolację oraz wytrzymałość. Więcej tutaj: https://pl.wikipedia.org/wiki/ACID.

MongoDB oparte jest o założenia CAP - ang. Consistency, Availability, and Partition co oznacza spójność, dostępność oraz tolerację na niedostępność wybranej partycji danych. Więcej: https://en.wikipedia.org/wiki/CAP_theorem.

**Skalowanie**

Skalowanie w przypadku bazy MySQL polega głównie na zwiększaniu pamięci dostępnej w ramach serwera bazy, co z reguły przy bardzo dużych systemach generuje znaczne koszty, a sam proces jest dość trudny i może wymagać czasowej niedostępności serwera bazy. Nie jest to jednak już tak duży problem, gdyż MySQL oraz większość komercyjnych silników baz danych umożliwia tworzenie klastra, w ramach krórego rozproszone maszyny mogą być wykorzystywane do obsługi jednej spójnej bazy.

MongoDB jest zoptymalizowane pod wykorzystyanie w popularnym aktualnie schemacie mikroserwisów, co pozwala rozbudowywać możliwości całego systemu poprzez dokładanie do niego nawet niewielkich maszyn (również wirtualnych), nazywanych tu węzłami, które podnoszą możliwości danej składowej systemu (obsługa bazy, serwer www).

Podejście z wykorzystaniem klastra ma niewątpliwą zaletę w postaci znacznego wzrostu bezpieczeństwa danych, gdyż dzięki mechanizmowi replikacji możliwe jest przechowywanie wielu kopii bazy i awaria jednego lub kilku węzłów (w zależności od konfiguracji) nie prowadzi do utraty danych oraz całkowitej niedostępności źródła danych a co najwyżej nieco dłuższy czas dostępu do danych przy dużym obciążeniu systemu. Dzięki takiemu podejściu możliwe jest też balansowanie obciążenia poszczególnych węzłów.

**Dostęp do danych i analiza**

W przypadku baz SQL-owych dostęp do danych odbywa się poprzez narzędzia umożliwiające połączenie z wybranym silnikiem bazy i komunikację poleceniami języka SQL. To jest wygodne i powszechne rozwiązanie a dzięki ustrukturyzowanym danym mamy też wiele narzędzi, które bez znajomości języka SQL również umożliwiają przeprowadzanie analizy danych przechowywanych w relacyjnych bazach (np. PowerBI, Tableau).

W przypadku baz NoSQL nie ma bezpośredniej możliwości przeprowadzenia takiej analizy. MongoDB posiada możliwość wysyłania zapytań o dokumenty w nim przechowywane, ale nie można porównać jego możliwości do języka SQL. 
Rozwiązaniem tego problemu może być import danych do magazynu SQL (data warehouse) lub wykorzystanie connectorów BI (ang. Business Intelligence), które umożliwiają łączenie danych z różnych źródeł (samo MongoDB nie obsługuje joinów) i przeprowadzenie analizy danych.
Istnieje też pojęcie witrualizacji danych, gdzię dzięki opracowaniu i przygotowaniu procesów ETL (Extract Transform Load) przygotowuje się dane, określa złączenia pomiędzy różnymi źródłami i można pracować na danych tak jak przez język SQL.
Istnieją również rozwiązania takie jak translatory, które tłumaczą polecenia języka SQL na polecenia zrozumiałe dla MongoDB np. Dremio.

Artykuł z linku poniżej (język angielski) podaje wiele przykładów zastosowania MongoDB w praktyce.  
Artykuł: https://www.upgrad.com/blog/mongodb-real-world-use-cases/


## 3. Uruchomienie lokalnej instancji MongoDB

MongoDB można używać jako serwisu w chmurze lub instalacji lokalnej. Na potrzeby tego laboratorium wybrana oztsła lokalna instalacja.
Na stronie https://www.mongodb.com/docs/manual/installation/ należy wybrać odpowiednią wersję dla systemu operacyjnego, pobrać i przeprowadzić proces instalacji.


Instrukcja instalacji dlasystemu Windows: https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-windows/


MongoDB dostarcza również narzędzie wizualne do zarządzania kolekcjami, dodawania, przeglądania i analizy danych w kolekcjach.


Po utworzeniu kolekcji trzeba dodać do niej jakieś przykładowe dane. Poniżej podano kilka linków do zbiorów danych w postaci plików json. 

Przykładowe dane do bazy: 
* https://media.mongodb.org/zips.json
* https://zaimki.pl/api
* api.openweathermap.org/data/2.5/weather?q=Krakow&appid={API}&units=metric


**Zapytania w MongoDB**

* https://www.mongodb.com/docs/compass/current/query/filter/


**Agregacje**

* https://www.mongodb.com/docs/compass/current/aggregation-pipeline-builder/

