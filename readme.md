# Zaawansowane Technologie Internetowe
## Przedmiot: bazy danych

## Opis logowania (PHPMyAdmin, putty, Workbench)

## 1. Tworzenie tabel i typy danych w bazach MySQL.

Polecenie `CREATE, ALTER, DROP` są poleceniami DDL (Data Definition Language - język definicji danych), czyli takimi, które manipulują strukturą dla danych w bazie. Możemy tę strukturę tworzyć, modyfikować i usuwać.
W tej części skupimy się na tworzeniu takich struktur i zaczniemy od tabel.

```sql
CREATE TABLE osoba (id int auto_increment PRIMARY KEY);
```


Lista typów danych : [typy danych MySQL](https://www.w3schools.com/sql/sql_datatypes.asp)

Złączanie tabel:
!['Złączanie tacel'](./sql_joins.jpg)