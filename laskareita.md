## viikko 2

### 10. Riippuvuuksien injektointi osa 3: Verkkokauppa

**Tämä ja seuraavat kaksi tehtävää tehdään palautusrepositorioon**

- lue [täältä](/tehtavat2/#tehtävien-palauttaminen) lisää tehtävien palautusrepositorioista

Tutustuimme viime viikon [tehtävissä 14-17](/tehtavat1#14-riippuvuuksien-injektointi-osa-1) riippuvuuksien injektointiin ja sovelsimme periaatetta yksikkötestauksen helpottamiseen.

Jos asia on päässyt unohtumaan, voit kerrata asian lukemalla [tämän](/riippuvuuksien_injektointi_python/).

[Kurssirepositorion]({{site.python_exercise_repo_url}}) hakemistossa _koodi/viikko2/verkkokauppa-1_ on yksinkertaisen verkkokaupan ohjelmakoodi

- Hae projekti kurssirepositoriosta
  - Järkevintä lienee että kloonaat kurssirepositorion paikalliselle koneellesi jos et ole sitä jo tehnyt, jos olet, niin pullaa repositorio ajantasalle
  - **Tämän jälkeen kannattaa kopioida projekti palautusrepositorioon, hakemiston viikko2 sisälle**
- Tutustu koodiin, piirrä luokkakaavio ohjelman rakenteesta
  - Luokkakaavioita ei tarvitse palauttaa
- Ohjelman luokista `Pankki`, `Varasto`, `Viitegeneraattori` ja `Kirjanpito` ovat sellaisia, että niistä on tarkoitus olla olemassa ainoastaan yksi olio. Tälläisiä ainutkertaisia olioita sanotaan **singletoneiksi**. Koodissa singletonit ovat toteutettu "klassisella tavalla", eli piilottamalla konstruktori ja käyttämällä staattista muuttujaa ja metodia säilömään ja palauttamaan luokan ainoa olio
  - Singleton on ns. [GoF-kirjan](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612) yksi alkuperäisistä suunnittelumalleista, lue lisää singletoneista esim. [täältä](https://sourcemaking.com/design_patterns/singleton)
  - Singleton ei ole erinäisistä syistä enää oikein muodissa, ja korvaamme sen seuraavassa tehtävässä
- Kuten huomaamme, on koodissa toivottoman paljon konkreettisia riippuvuuksia:
  - Varasto --> Kirjanpito
  - Pankki --> Kirjanpito
  - Kauppa --> Pankki
  - Kauppa --> Viitegeneraatori
  - Kauppa --> Varasto
- **Poista luokan `Kauppa` konkreettiset riippuvuudet** yllä mainittuihin luokkiin
  - Määrittele luokalle `Kauppa` sopiva konstruktori, jotta voit injektoida riippuvuudet
  - Riippuvuus luokkaan `Ostoskori` voi jäädä, sillä se on ainoastaan luokan Kauppa sisäisesti käyttämä luokka ja täten varsin harmiton
  - Muut riippuvuudet jätetään vielä
- Älä käytä luokan `Kauppa` sisällä enää konkreettisia luokkia `Varasto`, `Viitegeneraattori` ja `Pankki` vaan ainoastaan niitä vastaavia konstruktorin kautta saatuja olioita!
- **Muokkaa _index.py_-tiedoston `main`-funktiota**, siten että se luo kaupan seuraavasti:

```python
kauppa = Kauppa(
  Varasto.get_instance(),
  Pankki.get_instance(),
  Viitegeneraattori.get_instance()
)
```

- Asenna projektin riippuvuudet komennolla `poetry install`
- Varmista ohjelman toimivuus suorittamalla se virtuaaliympäristössä komennolla `python3 src/index.py`

### 11. Riippuvuuksien injektointi osa 4: ei enää singletoneja verkkokaupassa

- Singleton-suunnittelumallia pidetään [osittain ongelmallisena](http://rcardin.github.io/design/programming/2015/07/03/the-good-the-bad-and-the-singleton.html), poistammekin edellisestä tehtävästä singletonit
- **Poista** kaikista luokista `get_instance`-metodit ja staattinen `__instanssi`-muuttuja
- **Poista** rajapintojen ja riippuvuuksien injektoinnin avulla edellisen tehtävän jäljiltä jääneet riippuvuudet, eli
  - Varasto --> Kirjanpito
  - Pankki --> Kirjanpito
- **Muokkaa _index.py_-tiedoston `main`-funktiota** vastaamaan uutta tilannetta, eli suunnilleen muotoon:

```python
viitegeneraattori = Viitegeneraattori()
kirjanpito = Kirjanpito()
varasto = Varasto(kirjanpito)
pankki = Pankki(kirjanpito)
kauppa = Kauppa(varasto, pankki, viitegeneraattori)
```

Varmista ohjelman toimivuus suorittamalla se virtuaaliympäristössä komennolla `python3 src/index.py`.

### Yksinkertaistettu singeleton

Pythonin tapauksessa perinteisen singleton-suunnittelumallin mukaiset luokat tuottavat turhan monimutkaista koodia, joka on yksi syy niiden vähäiseen käyttöön. Esimerkiksi `Viitegeneraattori`-luokasta voisi yksinkertaisesti luoda olion, jota muut moduulit voivat käyttää:

```python
class Viitegeneraattori:
    def __init__(self):
        self._seuraava = 1

    def uusi(self):
        self._seuraava = self._seuraava + 1

        return self._seuraava


the_viitegeneraattori_olio = Viitegeneraattori()
```

Nyt muut moduulit voivat käyttää `the_viitegeneraattori_olio`-muuttujaan tallennettua oliota.

### 12. Riippuvuuksien injektointi osa 5: Verkkokauppa siistiksi

Edellisen tehtävän päätteeksi huomasimme, että `Kauppa`-luokan olion alustaminen vaatii melko paljon toimenpiteitä:

```python
viitegeneraattori = Viitegeneraattori()
kirjanpito = Kirjanpito()
varasto = Varasto(kirjanpito)
pankki = Pankki(kirjanpito)
kauppa = Kauppa(varasto, pankki, viitegeneraattori)
```

Korjataan tilanne antamalla riippuvuuksille oletusarvot.

**Tee seuraavat toimenpiteet:**

- Tallenna _viitegeneraattori.py_-tiedostossa muuttujaan `the_viitegeneraattori_olio` luokan `Viitegeneraattori` olio edellisen esimerkin tavoin
- Tallenna _kirjanpito.py_-tiedostossa muuttujaan `the_kirjanpito_olio` luokan `Kirjanpito` olio
- Muokkaa `Varasto`-luokkaa siten, että sen konstruktorin `kirjanpito`-parametrin arvo on oletusarvoisesti _kirjanpito.py_-tiedostossa määritelty `the_kirjanpito_olio`-muuttujan arvo. Parametrien oletuarvojen antaminen onnistuu seuraavasti:

```python
from kirjanpito import the_kirjanpito_olio

class Varasto:
    def __init__(self, kirjanpito=the_kirjanpito_olio):
            self._kirjanpito = kirjanpito
            # ...

    # ...
```

- Tallenna _varasto.py_-tiedostossa muuttujaan `the_varasto_olio` luokan `Varasto` olio. Huomaa, että olion voi alustaa ilman argumentteja (muodossa `Varasto()`), koska `kirjanpito`-parametrille on annettu oletusarvo.
- Tee sama `Pankki`-luokan konstruktorille ja tallenna `Pankki`-luokan olio muuttujaan `the_pankki_olio`
- Käytä `Kauppa`-luokan konstruktorissa `varasto`-, `pankki`- ja `viitegeneraattori`-parametrien oletusarvoina edellisissä askelissa määrittelemiäsi muuttujia
- **Muokkaa _index.py_-tiedoston `main`-funktiota** siten, että `Kauppa`-olion alustaminen ei käytä argumentteja:

```python
kauppa = Kauppa()
```

Huomaa, että luokalle voi silti halutessaan määritellä riippuvuudet argumentteina. Tämä on kätevää esimerkiksi testeissä:

```python
class PankkiStub:
    # ...

kauppa = Kauppa(pankki=PankkiStub())
```


## viikko 4


### 5. Ostoskori TDD-tekniikalla

Jatketaan verkkokaupan parissa. 

**Hae seuraavaksi [kurssirepositorion]({{site.python_exercise_repo_url}}) hakemistossa viikko4/tdd-ostoskori oleva projekti.**
- Kopioi projekti palatusrepositorioosi, hakemiston viikko4 sisälle.

Tässä tehtävässä muutamien luokkien toteutuksen logiikka on periaatteiltaan hieman erilainen kuin aiemmissa tehtävissä käsittelemässämme verkkokaupassa. Tehtävän fokuksessa on kolme luokkaa `Ostoskori`, `Ostos` ja `Tuote` joiden suhde on seuraava:

![](http://www.cs.helsinki.fi/u/mluukkai/otm2012/2.bmp)

Ostoskori siis sisältää _ostoksia_, joista jokainen vastaa yhtä tiettyä tuotetta.

Luokka `Tuote` on hyvin suoraviivainen. Tuotteesta tiedetään nimi, hinta ja varastosaldo (jota ei tosin käytetä mihinkään):

```python
class Tuote:
  def __init__(self, nimi: str, hinta: int):
      self._nimi = nimi
      self._hinta = hinta
      self._saldo = 0

  def hinta(self):
    return self._hinta

  def nimi(self):
    return self._nimi

  def __repr__(self):
      return f"{self._nimi} hinta {self._hinta} euroa"
```

Tuote siis kuvaa yhden tuotteen esim. _Valion Plusmaito_ tiedot (nimi, hinta ja varastosaldo, tuotteella voisi olla myös esim. kuvaus ja muita sitä luonnehtivia kenttiä).

**Ostoskoriin ei laiteta tuotteita vaan Ostoksia. Ostos viittaa tuotteeseen ja kertoo kuinka monesta tuotteesta on kysymys**. Eli jos ostetaan esim. 24 maitoa, tulee ostoskoriin Ostos-olio, joka viittaa Maito-tuoteolioon sekä kertoo, että tuotetta on korissa 24 kpl. `Ostos`-luokan koodi:

```python
from tuote import Tuote

class Ostos:
    def __init__(self, tuote: Tuote):
        self.tuote = tuote
        self._lukumaara = 1

    def tuotteen_nimi(self):
        return self.tuote.nimi()

    def muuta_lukumaaraa(self, muutos: int):
        self._lukumaara += muutos
        if self._lukumaara<0:
            self._lukumaara = 0

    def lukumaara(self):
        return self._lukumaara

    def hinta(self):
        return self._lukumaara * self.tuote.hinta()
```

Tehtävänäsi on ohjelmoida luokka `Ostoskori`.

Ostoskorin API:n eli metodirajapinta on seuraava (metodien rungoissa on `pass`-komennot, jotta Python-tulkki ei valittaisi syntaksivirheistä):

```python
from tuote import Tuote
from ostos import Ostos

class Ostoskori:
    def __init__(self):
        pass
        # ostoskori tallettaa Ostos-oliota, yhden per korissa oleva Tuote

    def tavaroita_korissa(self):
        pass
        # kertoo korissa olevien tavaroiden lukumäärän
        # jos koriin lisätty 2 kpl tuotetta "maito",
        #   tulee metodin palauttaa 2
        # jos korissa on 1 kpl tuotetta "maito" ja 1 kpl tuotetta "juusto",
        #   tulee metodin palauttaa 2

    def hinta(self):
        return 0
        # kertoo korissa olevien ostosten yhteenlasketun hinnan

    def lisaa_tuote(self, lisattava: Tuote):
        # lisää tuotteen
        pass

    def poista_tuote(self, poistettava: Tuote):
        # poistaa tuotteen
        pass

    def tyhjenna(self):
        pass
        # tyhjentää ostoskorin

    def ostokset(self):
        pass
        # palauttaa listan jossa on korissa olevat ostos-oliot
        # kukin ostos-olio siis kertoo mistä tuotteesta on kyse
        #   JA kuinka monta kappaletta kyseistä tuotetta korissa on
```

**Kerrataan vielä:** ostoskoriin lisätään Tuote-oliota metodilla `lisaa_tuote`. Ostoskori ei kuitenkaan talleta sisäisesti tuotteita vaan `Ostos`-luokan oliota (jotka viittaavat tuotteseen):

![](http://www.cs.helsinki.fi/u/mluukkai/otm2012/2.bmp)

Jos ostoskoriin laitetaan useampi kappale samaa tuotetta, päivitetään vastaavaa `Ostos`-oliota, joka muistaa kyseisen tuotteen lukumäärän.

**Ohjelmoi nyt ostoskori käyttäen [Test Driven Development](https://ohjelmistotuotanto-hy.github.io/osa3/#test-driven-development) -tekniikkaa.** Oikeaoppinen TDD etenee seuraavasti:

- Kirjoitetaan testiä sen verran että testi ei mene läpi. Ei siis luoda heti kaikkia luokan tai metodin testejä, vaan edetään yksi testi kerrallaan.
- Kirjoitetaan koodia sen verran, että testi saadaan menemään läpi. Ei yritetäkään heti kirjoittaa "lopullista" koodia.
- Jos huomataan koodin rakenteen menneen huonoksi (eli havaitaan koodissa esimerkiksi toisteisuutta tai liian pitkiä metodeja) refaktoroidaan koodin rakenne paremmaksi, ja huolehditaan koko ajan, että testit menevät edelleen läpi. Refaktoroinnilla tarkoitetaan koodin sisäisen rakenteen muuttamista siten, että sen rajapinta ja toiminnallisuus säilyy muuttumattomana.
- Jatketaan askeleesta 1

**Tee seuraavat testit ja aina jokaisen testin jälkeen testin läpäisevä koodi**. Jos haluat toimia oikean TDD:n hengessä, älä suunnittele koodiasi liikaa etukäteen, tee ainoastaan yksi askel kerrallaan ja paranna koodin rakennetta sitten kun koet sille tarvetta. Pidä _kaikki_ testit koko ajan toimivina. Eli jos jokin muutos hajottaa testit, älä etene seuraavaan askeleeseen ennen kuin kaikki testit menevät taas läpi.

Luokkia `Tuote` ja `Ostos` ei tässä tehtävässä tarvitse muuttaa ollenkaan.

_Lisää ja commitoi muutokset repositorioon jokaisen vaiheen jälkeen, anna kuvaava commit-viesti._

#### 1. Luodun ostoskorin hinta ja tavaroiden määrä määrä on 0.

Tehtäväpohjassa on yksi valmis testi

```python
class TestOstoskori(unittest.TestCase):
    def setUp(self):
        self.kori = Ostoskori()

    # step 1
    def test_ostoskorin_hinta_ja_tavaroiden_maara_alussa(self):
        self.assertEqual(self.kori.hinta(), 0)
```

Laajenna testiä siten, että se testaa myös tavaroiden määrän (metodin `tavaroita_korissa` paluuarvo). Kun testi on valmis, ohjelmoi ostoskoria sen verran että testi menee läpi. Tee ainoastaan minimaalisin mahdollinen toteutus, jolla saat testin läpi.

Lisää ja commitoi muutokset ja anna kuvaava commit-viesti.

#### 2. Yhden tuotteen lisäämisen jälkeen ostoskorissa on 1 tavara.

**Huom:** joudut siis luomaan testissäsi tuotteen jonka lisäät koriin:

```python
class TestOstoskori(unittest.TestCase):
    def setUp(self):
        self.kori = Ostoskori()

    # step 1
    def test_ostoskorin_hinta_ja_tuotteiden_maara_alussa(self):
        self.assertEqual(self.kori.hinta(), 0)
        # ...

    # step 2
    def test_yhden_tuotteen_lisaamisen_jalkeen_korissa_yksi_tavara(self):
        maito = Tuote("Maito", 3)
        self.kori.lisaa_tuote(maito)

        # ...
```

**Muistutus:** vaikka metodin `lisaa_tuote` parametrina on Tuote-olio, **ostoskori ei tallenna tuotetta** vaan luomansa Ostos-olion, joka "tietää" mistä tuotteesta on kysymys.

Lisää ja commitoi muutokset ja anna kuvaava commit-viesti.

#### 3. Yhden tuotteen lisäämisen jälkeen ostoskorin hinta on sama kuin tuotteen hinta.

Lisää ja commitoi muutokset.

#### 4. Kahden eri tuotteen lisäämisen jälkeen ostoskorissa on 2 tavaraa

Lisää ja commitoi muutokset.

#### 5. Kahden eri tuotteen lisäämisen jälkeen ostoskorin hinta on sama kuin tuotteiden hintojen summa

Lisää ja commitoi muutokset.

#### 6. Kahden saman tuotteen lisäämisen jälkeen ostoskorissa on 2 tavaraa

Lisää ja commitoi muutokset.

#### 7. Kahden saman tuotteen lisäämisen jälkeen ostoskorin hinta on sama kuin 2 kertaa tuotteen hinta

Lisää ja commitoi muutokset.

#### 8. Yhden tuotteen lisäämisen jälkeen ostoskori sisältää yhden ostoksen

tässä testataan ostoskorin metodia `ostokset`:

```python
    # step 8
    def test_yhden_tuotteen_lisaamisen_jalkeen_korissa_yksi_ostosolio(self):
        maito = Tuote("Maito", 3)
        self.kori.lisaa_tuote(maito)

        ostokset = self.kori.ostokset()

        # testaa että metodin palauttaman listan pituus 1
```

Lisää ja commitoi muutokset.

#### 9. Yhden tuotteen lisäämisen jälkeen ostoskori sisältää ostoksen, jolla sama nimi kuin tuotteella ja lukumäärä 1

Testin on siis tutkittava jälleen korin metodin ostokset palauttamaa listaa:

```python
    # step 9
    def test_yhden_tuotteen_lisaamisen_jalkeen_korissa_yksi_ostosolio_jolla_oikea_tuotteen_nimi_ja_maara(self):
        maito = Tuote("Maito", 3)
        self.kori.lisaa_tuote(maito)

        ostos = self.kori.ostokset()[0]

        # testaa täällä, että palautetun listan ensimmäinen ostos on halutunkaltainen.
```

Lisää ja commitoi muutokset.

#### 10. Kahden eri tuotteen lisäämisen jälkeen ostoskori sisältää kaksi ostosta

Lisää ja commitoi muutokset.

#### 11. Kahden saman tuotteen lisäämisen jälkeen ostoskori sisältää yhden ostoksen

Eli jos korissa on jo ostos "maito" ja koriin lisätään uusi "maito", tulee tämän jälkeen korissa olla edelleen vain yksi ostos "maito", lukumäärän tulee kuitenkin kasvaa kahteen.

Lisää ja commitoi muutokset.

#### 12. Kahden saman tuotteen lisäämisen jälkeen ostoskori sisältää ostoksen jolla sama nimi kuin tuotteella ja lukumäärä 2

Lisää ja commitoi muutokset.

#### 13. Jos korissa on kaksi samaa tuotetta ja toinen näistä poistetaan, jää koriin ostos jossa on tuotetta 1 kpl

Lisää ja commitoi muutokset.

#### 14. Jos koriin on lisätty tuote ja sama tuote poistetaan, on kori tämän jälkeen tyhjä

Tyhjä kori tarkoittanee että tuotteita ei ole, korin hinta on nolla ja ostoksien listan pituus nolla

Lisää ja commitoi muutokset.

#### 15. Metodi tyhjenna tyhjentää korin

Lisää ja commitoi muutokset.

Jos ostoskorissasi on mukana jotain ylimääräistä, refaktoroi koodiasi niin että kaikki turha poistuu. Erityisesti ylimääräisistä oliomuuttujista kannattaa hankkiutua eroon, tarvitset luokalle vain yhden oliomuuttujan, kaikki ylimääräiset tekevät koodista sekavamman ja vaikeammin ylläpidettävän.

Lisää ja commitoi mahdolliset muutokset.
