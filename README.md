# Security By Design

## Zadanie 1

### Identyfikacja problemu
Podczas analizy logów aplikacji (moduł Python) wykryto incydent bezpieczeństwa polegający na wycieku danych osobowych.
W logach zapisywane były one w formie niezaszyfrowanej.
![z1-1.png](.github/z1-1.png)

### Podjęte działania naprawcze
W celu wyeliminowania podatności wprowadzono mechanizm anonimizacji logów. Zmiany zostały zaimplementowane w pliku źródłowym `customers/models.py`.
 
Zastąpiono dotychczasową metodę odpowiedzialną za przygotowanie tekstowej reprezentacji danych klientów
```python
def __repr__(self):
    return f"Customer(ID: {self.id}, Name: {self.name}, City: {self.city}, Age: {self.age}, Pesel: {self.pesel}, Street: {self.street}, AppNo: {self.appNo})"

```
wersją, która anonimizuje wrażliwe informacje:
```python
def __repr__(self):
    return f"Customer(ID: {self.id}, Name: {'*'*len(self.name)}, City: {'*'*len(self.city)}, Age: {'*'*len(self.age)}, Pesel: {'*'*len(self.pesel)}, Street: {'*'*len(self.street)}, AppNo: {'*'*len(self.appNo)})"
```

### Weryfikacja
Ponowna weryfikacja logów potwierdziła skuteczność zabezpieczenia. Dane klientów nie są już zapisywane w logach aplikacji.
![z1-2.png](.github/z1-2.png)


## Zadanie 2: Weryfikacja wycieku sekretów

### Identyfikacja problemu
Przeprowadzono audyt repozytorium oraz historii zmian w celu wykrycia haseł, kluczy API lub innych sekretów zapisanych wprost w kodzie źródłowym lub plikach konfiguracyjnych projektu.

### Zidentyfikowane zagrożenia
W wyniku skanowania wykryto obecność 3 niezabezpieczonych kluczy w strukturze projektu. Sekrety znajdowały się w następujących plikach:
* `deployment.key`
* `deployment2.key`
* `awscredentials.json`

![z2-leaks.png](.github/z2.png)

Zauważono również duplikację kluczy pomiędzy plikami `deployment.key` a `deployment2.key`.

### Wnioski
Przeprowadzona weryfikacja potwierdziła, że znalezione ciągi znaków są rzeczywistymi sekretami, a nie fałszywymi alarmami (false-positives).
Wymagane jest ich natychmiastowe usunięcie z repozytorium oraz unieważnienie wyciekłych poświadczeń.

