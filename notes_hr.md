# 1. _extract temp to query_

Ekstrakcija privremenih varijabli u metode. Ovime kod postaje čitljiviji. Metoda može bit privatna, tako da zagađenje sučelja klase nije problem, a kod čitamo na prirodniji način.


_Methods longer than a line are a code smell_


# 2. _tell, don't ask_

Prosljeđuj naredbe objektima umjesto da im provjeravaš interno stanje.

`OrdersReport` provjerava polja koja pripadaju `Order`. Umjesto toga, `Order` bi to znanje sam trebao imati.


`Order#placed_between? (start_date, end_date)`

# 3. _data clump_

_združeni podaci_

`start_date` i `end_date` se metodama uvijek proslijeđuju zajedno. To je dobar znak da bi ih trebalo spojiti u objekt.

Većinom oklijevamo što se tiče ekstrakcije klasa, ali to stvarno može pomoći. Ako i nije za klasu, struct može biti koristan, a u skriptnim jezicima, kao JavaScript, čak i objektni literal može pojednostavniti stvari.

U našem slučaju, klasa čak ni nema ponašanje, a ipak poveća čitljivost i raumljivost koda.
Nešto što je  do sad bili implicitno je postalo eksplicitno. Sad **znamo** da početni i krajnji datum idu skupa.

Također, smanjuje povezanost (_coupling_) našeg koda.

# A. Još jedan primjer povezanosti (_coupling_)

Povezanost je stupanj do kojeg dvije komponente sustava ovise jedna o drugoj. Cilj je maksimalno smanjiti taj stupanj, kako bi, primjerice, komponentu A mogli mijenjati bez brige da će to utjecati na rad komponente B.

_Jedino vrijedno brige je kako lako/teško je mijenjati postojeći kod._

U našem primjeru imamo parametarsku povezanost. Kod na slici jako ovisi o tome kakav parametar prosljeđujemo ovoj funkciji. Ako proslijedimom, primjerice _nil_, kod puca jer `to_sentence` nije definiran.

# 4. `DateRange` je odlično mjesto za spremanje ponašanja

`Order#placed_between?` je bolje smješten u `DateRange#include?`

Primjer `Order#placed_between` je _feature envy_. Jedna klasa se previše bavi unutarnjim stvarima druge klase

Ne zaboraviti `(start_date..end_date).include? date # cover? date`

# B. Dobar savjet - pisati kod tako da se čita na isti način na koji bi se objašnjavao

"Total sales within date range are the total sales of the orders within range"

# 5. _null objects_

Nije uvijek preporučljivo, ali može pojednostaviti stvari.

Situacija: Klasa vraća defaultne vrijednosti za više svojstava, ako je neki parametar null.

Umjesto ručno dodijeljenih defaultova za svako svojstvo klase, jednostavno definiramo null objekt koji se dodijeli parametru automatski, ako mu ništa drugo nije dodijeljeno.

Za `Contact` imamo i `NullContact`

Primjena _tell, don't ask_ principa. Brisanje koda!

## Nedostatak null objekta

Sučelja pravog i null objekta moraju biti sinkronizirana. Ako dodamo/uklonimo _property_ s jednog, to treba napraviti i na drugom. To predstavlja vremenski trošak, ali u puno situacija vrijedi zbog pojednostavljenja koda.

Kolko često radimo stvari tipa

```
if current_user
    # bla bla bla
end
```

Null klase mogu komotn ostati u datoteci od klase roditelja, budući da ju samo roditelj privatno koristi.

Čak ju nije ni potrebno testirati, dok god je roditelj adekvatno testiran.


# 6 Oslanjanje na abstrakcije

Ove ideje smo uglavnom svijesni.

Koristimo EF i linq upite umjesto čistog SQLa za dohvat iz baze podataka. Koristimo neki http library umjesto slanja čistih bajtova na socket.

Ništa nas ne spriječava da s abstrakcijama idemo u više razina. Cjelina treba biti jednostavnija od sume dijelova.

U našem primjeru, koristimo eksterni gem za procesiranje naplata, koji je već sam po sebi poprilično dobra abstrakcija. Recimo da interno koristi HTTP, ali mi to ne znamo i to je super, jer to ne treba biti naša briga (osim činjenice da nam treba internet konekcija).

Ali, možemo i korak dalje.

Zašto bi korisnik morao znati da naplata za pretplatu uklučuje dohvat korisnika, dohvat njegovog braintree_id-a i onda kreiranje naplate za taj id?

Klasa `Refund` isto koristi BrainTree za svoje svrhe, što znači da obje klase pristupaju istom API-u, svaka na svoj način. Ako smo pisali testove za ove klase, vjerojatno smo umjesto pravog BrainTree gema, napravili _stubove_ koji ga samo oponašaju. Što ako se pravi Gem promijeni?

Što ako želimo promijeniti Gem koji koristimo za naplate?

Treba otvarati `User` klasu. To već samo po sebi zvuči čudno. Isto tako treba otvarati `Refund` klasu.

Sve to može biti abstrahirano iza još jednog sloja, pa napravimo `PaymentGateway`.

Općenito, ako više klasa, razni objekti, pristupaju istom API-u, onda bi taj pristup bilo dobro odijeliti u vlastiti sloj, izneđu tog API-a i naših klasa. Na taj način, sva interakcija s trećom stranom je lokalizirana u jednom objektu, koji onda možemo testirati, kao i koristiti ga za _stubbing_ u testovima ostatka aplikacije.

Nakon što imamo `PaymentGateway`, to otvara prednosti
* Testiranje samog gateway-a osigurava da on dobro radi
* Testiranje `User` i `Refund` klasa može se obaviti tako da umjesto gateway-a napravimo _stub_.
* Ako umjesto `BrainTree` želimo koristiti nešto drugo, treba promijeniti samo `PaymentGateway`

# Dobre mete za refactoring

1. _god objects_
- klase s kojima sve ima nekakvu interakciju.
- klasa koja predstavlja trenutnog korisnika je često takva klasa
- vrlo vjerojatno krši pravilo/princip jedne odgovornosti
- prvo pravilo klasa - moraju biti jako, jako male
- drugo pravilo klasa - moraju biti manje od toga

2. datoteke/moduli/klase koje se često mijenjaju
- ako se tako često mijenjaju, vjerojatno ga ne razumijemo

3. datoteke/moduli/klase u kojima se često javljaju bugovi
- bugovi vole društvo
- ako se javlja bug, imamo grešku u algoritmu, što znači da algoritam vjerojatno nije razumljiv, ili je prekompliciran da bi primjetili bug


# Neke knjige

* Clean Code: A Handbook of Agile Software
* Refactoring: Improving the Design of Existing Code
* Growing Object-Oriented SOftware Guided by Tests 