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


