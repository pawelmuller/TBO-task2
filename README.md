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



## Zadanie 3: Weryfikacja bezpieczeństwa bibliotek OpenSource

### Zakres działań
Przeprowadzono automatyczną analizę bezpieczeństwa (SCA) bibliotek zewnętrznych wykorzystywanych w projekcie w celu wykrycia znanych luk bezpieczeństwa.

### Wyniki audytu
![z3.png](.github/z3.png)

Przeskanowano 18 pakietów. Wykryto 12 podatności w 3 bibliotekach.

```
+==============================================================================+
|                                                                              |
|                               /$$$$$$            /$$                         |
|                              /$$__  $$          | $$                         |
|           /$$$$$$$  /$$$$$$ | $$  \__//$$$$$$  /$$$$$$   /$$   /$$           |
|          /$$_____/ |____  $$| $$$$   /$$__  $$|_  $$_/  | $$  | $$           |
|         |  $$$$$$   /$$$$$$$| $$_/  | $$$$$$$$  | $$    | $$  | $$           |
|          \____  $$ /$$__  $$| $$    | $$_____/  | $$ /$$| $$  | $$           |
|          /$$$$$$$/|  $$$$$$$| $$    |  $$$$$$$  |  $$$$/|  $$$$$$$           |
|         |_______/  \_______/|__/     \_______/   \___/   \____  $$           |
|                                                          /$$  | $$           |
|                                                         |  $$$$$$/           |
|  by pyup.io                                              \______/            |
|                                                                              |
+==============================================================================+
| REPORT                                                                       |
| checked 18 packages, using free DB (updated once a month)                    |
+============================+===========+==========================+==========+
| package                    | installed | affected                 | ID       |
+============================+===========+==========================+==========+
| jinja2                     | 3.1.2     | <3.1.3                   | 64227    |
+==============================================================================+
| Jinja is an extensible templating engine. Special placeholders in the        |
| template allow writing code similar to Python syntax. It is possible to      |
| inject arbitrary HTML attributes into the rendered HTML template,            |
| potentially leading to Cross-Site Scripting (XSS). The Jinja `xmlattr`       |
| filter can be abused to inject arbitrary HTML attribute keys and values,     |
| bypassing the auto escaping mechanism and potentially leading to XSS. It may |
| also be possible to bypass attribute validation checks if they are           |
| blacklist-based.                                                             |
+==============================================================================+
| jinja2                     | 3.1.2     | <3.1.4                   | 71591    |
+==============================================================================+
| Jinja is an extensible templating engine. The `xmlattr` filter in affected   |
| versions of Jinja accepts keys containing non-attribute characters. XML/HTML |
| attributes cannot contain spaces, `/`, `>`, or `=`, as each would then be    |
| interpreted as starting a separate attribute. If an application accepts keys |
| (as opposed to only values) as user input, and renders these in pages that   |
| other users see as well, an attacker could use this to inject other          |
| attributes and perform XSS. The fix for CVE-2024-22195 only addressed spaces |
| but not other characters. Accepting keys as user input is now explicitly     |
| considered an unintended use case of the `xmlattr` filter, and code that     |
| does so without otherwise validating the input should be flagged as          |
| insecure, regardless of Jinja version. Accepting _values_ as user input      |
| continues to be safe.                                                        |
+==============================================================================+
| jinja2                     | 3.1.2     | <3.1.5                   | 76378    |
+==============================================================================+
| An oversight in how the Jinja sandboxed environment detects calls to         |
| str.format allows an attacker who controls the content of a template to      |
| execute arbitrary Python code. To exploit the vulnerability, an attacker     |
| needs to control the content of a template. Whether that is the case depends |
| on the type of application using Jinja. This vulnerability impacts users of  |
| applications which execute untrusted templates. Jinja's sandbox does catch   |
| calls to str.format and ensures they don't escape the sandbox. However, it's |
| possible to store a reference to a malicious string's format method, then    |
| pass that to a filter that calls it. No such filters are built-in to Jinja,  |
| but could be present through custom filters in an application. After the     |
| fix, such indirect calls are also handled by the sandbox.                    |
+==============================================================================+
| jinja2                     | 3.1.2     | <3.1.6                   | 75976    |
+==============================================================================+
| Prior to 3.1.6, an oversight in how the Jinja sandboxed environment          |
| interacts with the |attr filter allows an attacker that controls the content |
| of a template to execute arbitrary Python code. To exploit the               |
| vulnerability, an attacker needs to control the content of a template.       |
| Whether that is the case depends on the type of application using Jinja.     |
| This vulnerability impacts users of applications which execute untrusted     |
| templates. Jinja's sandbox does catch calls to str.format and ensures they   |
| don't escape the sandbox. However, it's possible to use the |attr filter to  |
| get a reference to a string's plain format method, bypassing the sandbox.    |
| After the fix, the |attr filter no longer bypasses the environment's         |
| attribute lookup. This vulnerability is fixed in 3.1.6.                      |
+==============================================================================+
| jinja2                     | 3.1.2     | >=3.0.0a1,<3.1.5         | 74735    |
+==============================================================================+
| A vulnerability in the Jinja compiler allows an attacker who can control     |
| both the content and filename of a template to execute arbitrary Python      |
| code, bypassing Jinja's sandbox protections. To exploit this vulnerability,  |
| an attacker must have the ability to manipulate both the template's filename |
| and its contents. The risk depends on the application's specific use case.   |
| This issue affects applications that render untrusted templates where the    |
| attacker can determine the template filename, potentially leading to severe  |
| security breaches.                                                           |
+==============================================================================+
| werkzeug                   | 2.3.7     | <2.3.8                   | 62019    |
+==============================================================================+
| Werkzeug 3.0.1 and 2.3.8 include a security fix: Slow multipart parsing for  |
| large parts potentially enabling DoS attacks.                                |
| https://github.com/pallets/werkzeug/commit/b1916c0c083e0be1c9d887ee2f3d69692 |
| 2bfc5c1                                                                      |
+==============================================================================+
| werkzeug                   | 2.3.7     | <3.0.3                   | 71594    |
+==============================================================================+
| Werkzeug is a comprehensive WSGI web application library. The debugger in    |
| affected versions of Werkzeug can allow an attacker to execute code on a     |
| developer's machine under some circumstances. This requires the attacker to  |
| get the developer to interact with a domain and subdomain they control, and  |
| enter the debugger PIN, but if they are successful it allows access to the   |
| debugger even if it is only running on localhost. This also requires the     |
| attacker to guess a URL in the developer's application that will trigger the |
| debugger.                                                                    |
+==============================================================================+
| werkzeug                   | 2.3.7     | <3.0.6                   | 73969    |
+==============================================================================+
| Affected versions of Werkzeug are vulnerable to Path Traversal (CWE-22) on   |
| Windows systems running Python versions below 3.11. The safe_join() function |
| failed to properly detect certain absolute paths on Windows, allowing        |
| attackers to potentially access files outside the intended directory. An     |
| attacker could craft special paths starting with "/" that bypass the         |
| directory restrictions on Windows systems. The vulnerability exists in the   |
| safe_join() function which relied solely on os.path.isabs() for path         |
| validation. This is exploitable on Windows systems by passing paths starting |
| with "/" to safe_join(). To remediate, upgrade to the latest version which   |
| includes additional path validation checks.                                  |
| NOTE: This vulnerability specifically affects Windows systems running Python |
| versions below 3.11 where ntpath.isabs() behavior differs.                   |
+==============================================================================+
| werkzeug                   | 2.3.7     | <3.0.6                   | 73889    |
+==============================================================================+
| Affected versions of Werkzeug are potentially vulnerable to resource         |
| exhaustion when parsing file data in forms. Applications using               |
| 'werkzeug.formparser.MultiPartParser' to parse 'multipart/form-data'         |
| requests (e.g. all flask applications) are vulnerable to a relatively simple |
| but effective resource exhaustion (denial of service) attack. A specifically |
| crafted form submission request can cause the parser to allocate and block 3 |
| to 8 times the upload size in main memory. There is no upper limit; a single |
| upload at 1 Gbit/s can exhaust 32 GB of RAM in less than 60 seconds.         |
+==============================================================================+
| werkzeug                   | 2.3.7     | <3.1.4                   | 82196    |
+==============================================================================+
| Affected versions of the Werkzeug package are vulnerable to Denial of        |
| Service (DoS) due to improper handling of Windows special device names in    |
| the safe_join function. In Werkzeug versions before 3.1.4, safe_join permits |
| path segments such as “CON” or “AUX” to pass validation, allowing            |
| send_from_directory to construct a path that resolves to a Windows device    |
| file, which opens successfully but then blocks indefinitely when read.       |
+==============================================================================+
| werkzeug                   | 2.3.7     | <=2.3.7                  | 71595    |
+==============================================================================+
| Werkzeug is a comprehensive WSGI web application library. If an upload of a  |
| file that starts with CR or LF and then is followed by megabytes of data     |
| without these characters: all of these bytes are appended chunk by chunk     |
| into internal bytearray and lookup for boundary is performed on growing      |
| buffer. This allows an attacker to cause a denial of service by sending      |
| crafted multipart data to an endpoint that will parse it. The amount of CPU  |
| time required can block worker processes from handling legitimate requests.  |
+==============================================================================+
| healpy                     | 1.8.0     | <=1.16.6                 | 61774    |
+==============================================================================+
| Healpy 1.16.6 and prior releases ship with a version of 'libcurl' that has a |
| high-severity vulnerability.                                                 |
+==============================================================================+
```

### Najpoważniejsze zagrożenie
* **Jinja2 (RCE, XSS):** Krytyczne ryzyko obejścia mechanizmu sandbox i wykonania dowolnego kodu, jeśli aplikacja przetwarza niezaufane szablony.
* **Werkzeug (DoS):** Wysokie ryzyko ataku typu *Denial of Service* (wyczerpanie pamięci RAM) poprzez spreparowane żądania HTTP.
* **Healpy:** Podatność o wysokiej krytyczności w zależnej bibliotece `libcurl`.

### Ocena ryzyka
Poziom ryzyka globalnego: **WYSOKI**

1.  **Krytyczność Jinja2 (RCE):** Ryzyko jest **ZALEŻNE OD KONTEKSTU**.
    * Jeżeli aplikacja renderuje szablony tworzone przez użytkowników (np. systemy CMS, e-maile edytowane przez klienta), ryzyko jest **KRYTYCZNE** (możliwość przejęcia serwera).
    * Jeżeli szablony są statyczne i kontrolowane tylko przez programistów, ryzyko jest **Niskie**.
2.  **Krytyczność Werkzeug (DoS):** Ryzyko jest **WYSOKIE**.
    * Podatność na ataki DoS jest łatwa do wykorzystania w każdej aplikacji publicznie dostępnej (Flask domyślnie używa Werkzeug). Atak nie wymaga uwierzytelnienia i może skutecznie wyłączyć usługę.
3.  **Krytyczność Werkzeug (Debugger RCE):** Ryzyko jest **ŚREDNIE/NISKIE**.
    * Wymaga, aby debugger był włączony (`debug=True`) oraz interakcji dewelopera, co w środowisku produkcyjnym nie powinno mieć miejsca.

### Rekomendacja:
Natychmiastowa aktualizacja bibliotek do wersji bezpiecznych (lub najnowszych dostępnych):
* `jinja2` $\rightarrow$ **3.1.6+**
* `werkzeug` $\rightarrow$ **3.1.4+**
* `healpy` $\rightarrow$ **najnowsza stabilna**
