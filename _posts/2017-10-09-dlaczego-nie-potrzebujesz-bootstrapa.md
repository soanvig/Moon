---
layout: post
title: Dlaczego nie potrzebujesz Bootstrapa?
tags: [CSS]
comments: true
excerpt: Nadużywanie i wszechobecność Bootstrapa to rak trawiący sieć. A dlaczego - dowiesz się w tym wpisie.
---

<!-- TOC -->

- [Wstęp](#wst%C4%99p)
- [Co jest nie tak z Bootstrapem?](#co-jest-nie-tak-z-bootstrapem)
  - [Waga](#waga)
    - [Rozwiązanie](#rozwi%C4%85zanie)
  - [Mieszanie warstw treści i wyglądu](#mieszanie-warstw-tre%C5%9Bci-i-wygl%C4%85du)
    - [Rozwiązanie](#rozwi%C4%85zanie-1)
  - [Reguły !important](#regu%C5%82y-important)
    - [Rozwiązanie](#rozwi%C4%85zanie-2)
  - [Bootstrap jest wszędzie](#bootstrap-jest-wsz%C4%99dzie)
    - [Rozwiązanie](#rozwi%C4%85zanie-3)
  - [Polecanie Bootstrapa na początku nauki](#polecanie-bootstrapa-na-pocz%C4%85tku-nauki)
  - [Zapominanie o jakości HTML](#zapominanie-o-jako%C5%9Bci-html)
- [Własnoręcznie stworzone alternatywy dla komponentów BS](#w%C5%82asnor%C4%99cznie-stworzone-alternatywy-dla-komponent%C3%B3w-bs)
  - [Grid](#grid)
    - [Ustawienie kolumn obok siebie](#ustawienie-kolumn-obok-siebie)
      - [Rozwiązanie problemu float – pseudoelement](#rozwi%C4%85zanie-problemu-float-%E2%80%93-pseudoelement)
      - [Rozwiązanie problemu float – overflow: hidden](#rozwi%C4%85zanie-problemu-float-%E2%80%93-overflow-hidden)
      - [Rozwiązanie problemu float - display: flow-root](#rozwi%C4%85zanie-problemu-float---display-flow-root)
    - [Ustawienie szerokości kolumn](#ustawienie-szeroko%C5%9Bci-kolumn)
    - [Breakpointy](#breakpointy)
    - [Grid: słowem zakończenia](#grid-s%C5%82owem-zako%C5%84czenia)
  - [Modal](#modal)
    - [Podstawowa struktura](#podstawowa-struktura)
    - [Zmiana stanu modala](#zmiana-stanu-modala)
    - [Sterowanie modalem](#sterowanie-modalem)
    - [Co można ulepszyć](#co-mo%C5%BCna-ulepszy%C4%87)
- [Podsumowanie](#podsumowanie)

<!-- /TOC -->

## Wstęp

[Bootstrap](http://getbootstrap.com/) (BS) - biblioteka/framework HTML/CSS. Narzędzie, które umożliwia szybkie tworzenie witryn przy użyciu gotowych klas. Rak trawiący sieć. Możesz też przeskoczyć od razu do [podsumowania](#podsumowanie).

![]({{ site.url }}/assets/images/dlaczego-nie-potrzebujesz-bootstrapa/bootstrap.png)

## Co jest nie tak z Bootstrapem?

Sama idea Bootstrapa (czy w praktyce jakiejkolwiek biblioteki HTML/CSS) jest jak najbardziej w porządku:

> Bootstrap is an open source toolkit for developing with HTML, CSS, and JS. Quickly prototype your ideas or build your entire app with our Sass variables and mixins, responsive grid system, extensive prebuilt components, and powerful plugins built on jQuery. 

I to jest okej! Zwracam szczególną uwagę na fragment o "szybkim prototypowaniu pomysłów". Bootstrap faktycznie umożliwia szybkie prototypowanie stron. Ale prototyp to nie strona nadająca się na produkcję.

Poniżej opisane "wady" BS często dotyczą ogólnie całego środowiska BS niż tylko biblioteki *per se*.
Warto też zwrócić uwagę na to, że podobne uwagi można mieć do wielu bibliotek CSS (np. Foundation). Jeśli znajdujesz jakieś nowe rozwiązanie możesz sprawdzić je pod kątem rzeczy wymienionych w tym artykule jako wady BS.

### Waga

Zminifikowany CSS Bootstrapa 4.0.0-beta to 120kB. Waga zminifikowanego JS Bootstrapa to 50kB. Razem 170kB.
Ale JS Bootstrapa wymaga jQuery (70kB) i biblioteki Popper (20kB). Łącznie mamy **260kB w czterech requestach**.

260kB, a do tego dochodzą chętnie używane pluginy do BS i jQuery, które potrafią nawet **kilkukrotnie podbić tę wartość** (np. template'y/biblioteki/frameworki oparte na BS) i dodawać kolejne requesty. Każdy kolejny request to dodatkowy czas ładowania (pomijając samą wagę odpowiedzi!).

Dodać do tego możemy jeszcze wagę naszego własnego CSS, który napiszemy (choć ze względu na BS będzie oczywiście mniejsza niż normalnie).

Nie byłoby w tej wadze nic złego, gdyby przeciętny webdeveloper wykorzystał wszystkie oferowane przez Bootstrapa klasy oraz przez jQuery metody. Jednak nie oszukujmy się – tak nie jest i prawdopodobnie nigdy nie będzie. W każdym innym (czyli w rzeczywistości – w każdym) przypadku zasysamy **zbędne kilobajty** kodu, którego nigdy przeglądarka klienta nie wykorzysta. Zbędne kilobajty dopuszczalne są w fazie rozwijania produktu (*development*), ale nie w momencie udostępniania go klientowi (*production*).

Rozwiązania skrojone na miarę są **zawsze lepsze**, nie muszę tłumaczyć czemu. Oczywiście w przypadku kompletnych aplikacji webowych niemożliwym jest poświęcenie czasu na stworzenie wszystkich potrzebnych narzędzi (np. narzędzi JavaScript typu *Owl Carousel* albo *Cropper*), ponieważ są zwyczajnie zbyt złożone (a niektórzy webdeveloperzy wręcz nie są w stanie zrobić takich narzędzi dobrze). Wtedy zassanie takich bibliotek wydaje się dopuszczalne. Już samo wykorzystywanie jQuery przez Bootstrapa jest kontrowersyjne – większość stron nie wykorzysta w pełni możliwości jQuery.

A bądźmy między sobą szczerzy: cóż takiego oferuje Bootstrap poza zupełnie podstawowymi regułami CSS? Kilka fajnych skrypcików w JS i tyle. I na dodatek do tego jeszcze wymaga jQuery. A zazwyczaj [z jQuery można zrezygnować](http://youmightnotneedjquery.com/).

Oczywiście, można stworzyć własne kompilacje BS i jQ zawierające tylko to, co faktycznie jest potrzebne. I oczywiście – prawie nikt tego nie robi (choć sytuacja wydaje się być lepsza przy dużych aplikacjach).

#### Rozwiązanie

1. Zamiast BS stosować własny kod CSS.
2. Starać się zrezygnować z jQuery na rzecz czystego JSa (zob. http://youmightnotneedjquery.com/).
3. Stosować BS co najwyżej dla panelów administracyjnych (a nie dla przeciętnego klienta strony) lub przy prototypowaniu.
4. W ostateczności stworzyć własne kompilacje BS i jQ dopasowane do swoich wymagań.

### Mieszanie warstw treści i wyglądu

W przypadku stron internetowych treścią (widokiem, ang. *view*) nazywamy kod HTML (czyli język opisu dokumentu), a wyglądem kod CSS. Gdy mówimy o innych rozwiązaniach (aplikacje okienkowe na Windowsa, Qt, GTK, frameworki backendowe) zawsze mówimy o rozdzielaniu warstw treści, logiki i wyglądu. Teksty często wyłączamy do osobnych plików (zaleta: łatwa translacja), logikę oddzielamy od treści (aby móc wykorzystać treść w wielu miejscach i zachować porządek w aplikacji), całą strukturę plików grupujemy w katalogi i tak dalej.

Bootstrap tymczasem (oferując m.in. klasy z kategorii *Utilities*) **zachęca do mieszania warstw treści i wyglądu**. W jaki sposób?

Konwencje Bootstrapa sugerują wykorzystywanie klas typu `.mt-5` (`margin-top: 5px;`) czy `.float-left` (`float: left;`). Oh, ileż się naczytałem na forach jakie to style inline są złe, a jednocześnie obok polecano Bootstrap. A czym się różni:

```html
<span class="float-left"></span>
```

od

```html
<span style="float: left;"></span>
```

Oczywiście niczym.

Po pierwsze stosując powyższe style traci się panowanie nad tym, które style aplikowane są w plikach HTML, a które w plikach CSS. Po drugie brudzi się kod – zamiast opisywać w HTML, czym są dane elementy (opis elementu za pośrednictwem klas, zob. BEM), opisuje się ich wygląd.

#### Rozwiązanie

1. Nie korzystać z Bootstrapa.
2. W przypadku używania Bootstrapa nie stosować klas, których celem jest tylko i wyłącznie nie pisanie kodu CSS (między innymi klas *Utilities*).
3. Do opisu elementów HTML używać nie stylów inline (znanych jako "klasy"), tylko klas stworzonych według metodologii [BEM](http://getbem.com/introduction/) (która wcale nie ma zastosowania tylko do HTML!) – czyli nazywania elementów według tego, *czym są*.

### Reguły !important

Po otworzeniu pliku CSS Bootstrapa i wykonania opcji "Szukaj" dla zapytania "!important" otrzymałem… 1000+ wyników.

![Obrazek przedstawiający ponad 1000 wyników dla zapytania important w arkuszu Bootstrapa]({{ site.url }}/assets/images/dlaczego-nie-potrzebujesz-bootstrapa/important.jpg)

Co to oznacza? Ogólnie stosowania reguł `!important` powinno się unikać, ponieważ czynią praktycznie niemożliwym ich nadpisanie – reguły nadpisujące również wymagają `!important`. Możemy się spodziewać, że domyślny wygląd Bootstrapa nie będzie nam odpowiadał, więc zaczniemy pisać własny kod CSS, który będzie musiał korzystać z tej reguły, żeby nadpisać niektóre style Bootstrapa. I wtedy zaczyna się zabawa w specyficzność w CSS, z którą *gros* osób – szczególnie początkujących – ma problem. Już pomijając fakt związany z frustracją i "Mój CSS nie działa!".

#### Rozwiązanie

1. Nie korzystać z Bootstrapa.
2. Korzystać z domyślnych stylów Bootstrapa i nie pisać żadnego kodu CSS.

### Bootstrap jest wszędzie

Bootstrap **jest wszędzie**. Jest pakowany do panelów administracyjnych (według mnie użycie dozwolone), stron one-page, witryn firmowych, aplikacji internetowych, do – **o zgrozo** – portfolio webdeveloperów, a konkretnych przykładów można by wymieniać w nieskończoność.

Ktoś powie "ot, popularne narzędzie". Otóż to nie tylko popularne narzędzie, ale w pewnym sensie popularny szablon. http://adventurega.me/bootstrap/ i tyle w temacie.

Czym może się pochwalić webmaster, który swoją prywatną stronę-portfolio stawia na Bootstrapie? Niczym.

#### Rozwiązanie

1. Współpracować z grafikiem.
2. Zakupić szablon i samemu go okodować.
3. Nauczyć się tworzyć wygląd w dobrym stylu.
4. Skopiować wygląd Bootstrapa i okodować go samodzielnie.

### Polecanie Bootstrapa na początku nauki

Uczestnicząc w życiu pewnego forum poświęconemu między innymi front-endowi, często widzę, że jako remedium na **normalne problemy na początku nauki** poleca się Bootstrapa. Nie umiesz zrobić RWD? Bootstrap. Nie umiesz zrobić modala? Bootstrap ma modal. Dropdown? Bootstrap. Tooltip? Bootstrap.

Faktycznie, Bootstrap zawiera bardzo dużo przydatnych komponentów, ale nie można polecać tego typu rozwiązań na początku nauki. Należy pokazać drogę, którą można podążać, wskazać możliwe rozwiązanie (bez podawania gotowców). Dzięki samodzielnej pracy i samodzielnemu rozwiązywaniu problemów (czasem zajmujących wiele dni) człowiek uczy się **najszybciej**. Cóż komu po takim front-endowcu, który potrafi używać Bootstrapa, ale napisanie bardziej złożonego kodu w CSS jest dla niego katorgą, o JS (jQ) wie tyle, ile mówi dokumentacja BS (czyli jak używać komponentów), a połowa jego kodu to kopiuj-wklej z tejże dokumentacji? I nie, **nie przesadzam**. Tak jest, co widać po kolejnych stronach opartych na BS i pytaniach (i odpowiedziach do nich) na forach. Ba, taki front-endowiec myśli nieraz, że już wie wszystko o front-endzie. To z kolei drażni te osoby, które faktycznie wiedzą nieco więcej.

### Zapominanie o jakości HTML

Gdy przyjrzymy się stronom opartym na BS widać od razu, że ich developerzy lekką ręką tworzą kolejne zagnieżdżone w sobie divy (ze stylami inline) i zapominają o czymś takim jak [semantyczny kod](https://tutorials.comandeer.pl/html5-blog.html). Pierwsza rzecz prowadzi do mniej czytelnego, trudnego w utrzymaniu i ciężkiego kodu HTML, a druga do utrudniania pracy czytnikom ekranowym, robotom wyszukiwarek i współpracownikom. Tak jest – semantyczny kod jest łatwiejszy w utrzymaniu przy współpracy wielu developerów!

Apeluję: **nie twórzmy zbędnych tagów HTML**. 90% przypadków można uprościć tylko przez **umiejętne stosowanie CSS** (rzecz niemożliwą do nauki, jeśli [od jej początku poleca się Bootstrap](#polecanie-bootstrapa-na-początku-nauki)) i pseudoelementów, a dzięki temu uzyskujemy kod czytelniejszy i lżejszy!

Nie jestem w stanie dokładnie podać przyczyny tego zjawiska, ale wynika prawdopodobnie ono z tego, że w swych konwencjach BS wymaga stosowania kilku rodzajów rodziców-kontenerów, aby osiągnąć wymagany efekt. Przyzwyczajony do czegoś takiego webmaster stosuje następnie tę metodologię w samodzielnie napisanym kodzie.

## Własnoręcznie stworzone alternatywy dla komponentów BS

Często argumentem za używaniem BS są komponenty typu grid lub modal. Można jednak napisać ich ekwiwalenty (czasem nawet lepsze od oryginału) albo posługując się bezpośrednio kodem BS, albo poniższymy radami. Unikniemy wtedy wymogu zasysania BS z jego zależnościami.

Jeśli w poniższej rozpisce brakuje komponentu, który by Was interesował, proszę o napisanie w komentarzu.

### Grid

Chciałbym zacząć od tego, że przy stosowaniu tylko i wyłącznie BEM do nazywania elementów (tak, że każdy element posiada swój opis w postaci klas) nie ma potrzeby używania takiego modułu jak "grid" – faktyczne style ustawiające elementy w kolumny mogą pojawić się jako część klas bezpośrednio dotyczących danego elementu (BEM). Ten rozdział jest wyłącznie lekcją, jak coś takiego wykonać.

Grid Bootstrapa 4 oparty jest na flexboxie, nieraz trudnym w zrozumieniu początkującym, więc spróbuję omówić go na podstawie BS 3, gdzie zbudowany był na floatach. Warto wiedzieć, że zastosowanie flexboxu znacznie upraszcza i zmniejsza kod CSS.

Oto, jak łatwo osiągnąć grid podobny do tego w BS: 3 kolumny (25%/50%/25%), które przy szerokości ekranu "800px" przeskakują do nowych linii. Rozwiązanie skrojone na miarę:

```html
<div class="parent">
    <div class="grid-col grid-col--1"></div>
    <div class="grid-col grid-col--2"></div>
    <div class="grid-col grid-col--1"></div>
</div>
```

```css
.parent {
    overflow: hidden;
}

.grid-col {
    float: left;
}

.grid-col--1 {
    width: 25%;
}

.grid-col--2 {
    width: 50%;
}

@media all and (max-width: 800px) {
    .grid-col {
        width: 100%;
        float: none;
    }
}
```

A teraz zapraszam do omówienia tego kodu:

#### Ustawienie kolumn obok siebie

W BS3 grid wykorzystuje klasy kolumn `.col`, które są właściwie najważniejsze i na nich się skupimy. Po pierwsze przejdźmy na BEM (element-dziecko oznaczam przez pojedynczą podłogę, a modyfikator przez podwójnego dasha. Myślnik oznacza złożoną nazwę bloku):

```css
.grid-col {}
```

Dlaczego `col` nie jest dzieckiem `grid` (`.grid_col`)? Ponieważ rzadko kiedy będziemy musieli zastosować klasę `.grid` (praktycznie nigdy). Nadal jednak warto utrzymać namespace i oznaczyć, że `col` to kolumna, ale modułu `grid`. Stąd `.grid-col`.

Po pierwsze kolumny muszą ustawić się obok siebie. Dlatego nadajemy im `float: left;`.

**Rada:** nadanie kolumnom `float: right` odwróci ich kolejność (w porównaniu z kolejnością wynikającą z HTML).

```css
.grid-col {
    float: left;
}
```

![Obrazek przedstawiający ustawienie elementów obok siebie po zastosowaniu float: left]({{ site.url }}/assets/images/dlaczego-nie-potrzebujesz-bootstrapa/grid-1.png)

Jak widać, elementy ustawiły się obok siebie. Zastosowanie float powoduje jednak pewien problem, który widać po otworzeniu inspektora i najechaniu na rodzica kolumn:

![Obrazek przedstawiający zerową wysokość rodzica kolumn]({{ site.url }}/assets/images/dlaczego-nie-potrzebujesz-bootstrapa/grid-2.png)

Rodzic (`div`) posiada zerową wysokość, pomimo tego, że kolumny wyraźnie tę wysokość posiadają (przykładowe 200px). Nadanie np. koloru tła dla tego rodzica nie przyniesie efektu. Kolumny powinny rozciągać rodzica, a tak się nie dzieje, ponieważ zastosowaliśmy float. Float powoduje wyciągnięcie elementu z normalnego obiegu (tak samo się dzieje przy `position: absolute`). W tym przypadku rodzic nawet nie wie, że się w nim znajdują jakieś dzieci - stąd zerowa wysokość.

W CSS istnieje reguła `clear`, która powoduje, że element z tą regułą nie będzie chciał sąsiadować z elementami posiadającymi `float`. Nas interesuje wartość `clear: both;`.

Dobrze, możemy więc dodać element do rodzica, który nie będzie floatował, a będzie miał `clear: both`. Jeśli umieścimy go na końcu rodzica, to rozciągnie nam rodzica na oczekiwaną wysokość. **Błąd!** Tworzymy bowiem pusty tag HTML, a pusty tag to zbędny tag, ponieważ nie zawiera treści. Jeśli coś nie zawiera treści to z punktu widzenia HTML nie ma sensu. Jeśli patrzymy na niego tylko przez przyzmat CSS to oznacza, że powinno się jego rolę jakoś wepchnąć do CSSa. Na szczęście się da:

##### Rozwiązanie problemu float – pseudoelement

Zamiast tworzyć element w HTML, możemy ten element stworzyć w CSS z wykorzystaniem [pseudoelementów](http://www.kurshtml.edu.pl/css/pseudoelementy.html). Ponieważ chcemy, aby ten element był stworzony na końcu rodzica zastosujemy `::after`.

Nadaję tutaj rodzicowi **przykładową** klasę `.parent`.

```css
.parent::after {
    content: '';
    clear: both;
    display: block;
}
```

![Obrazek przedstawiający rozszerzenie się rodzica po stworzeniu pseudoelementu]({{ site.url }}/assets/images/dlaczego-nie-potrzebujesz-bootstrapa/grid-3.png)

Oprócz `content`, bez którego pseudoelement nie istnieje, należało zastosować `display: block`, aby element chciał się wyświetlić **za kolumnami**, a nie obok nich. Rozciągnięcie nastąpi tylko wtedy, jeśli ustawi się za nimi, a domyślnie pseudolement jest liniowy (`inline`).

##### Rozwiązanie problemu float – overflow: hidden

Drugim sposobem jest ustawienie `overflow: hidden;` dla rodzica. Dzięki temu sama dolna krawędź rodzica będzie posiadała cechę `clear: both` (niemożliwa rzecz do uzyskania przez CSS w inny sposób).

Wadą tego rozwiązania jest to, że jeśli posiadamy elementy wystające z rodzica (w czasach dzisiejszych layoutów rzadki przypadek), to zostaną one przycięte do rodzica.

Zaletą jest to, że możemy zaoszczędzony pseudoelement wykorzystać do czegoś innego.

##### Rozwiązanie problemu float - display: flow-root

Ostatnim rozwiązaniem, które mogę polecić, to nadanie `display: flow-root` dla kolumny. Warto jednak mieć na uwagę [wsparcie](http://caniuse.com/#search=flow-root) tej własności.

#### Ustawienie szerokości kolumn

Zastosowanie float na kolumnach powoduje, że domyślnie dostosowują one swoją szerokość do treści. Kolumny powinny jednak dostosowywać swoje wymiary do szerokości rodzica i zajmować jego określoną część (np. BS dzieli rodzica na 12 części). Sprawa wyjątkowo prosta, trzeba tylko policzyć ile to jest 1/12 z 100% (kalkulator, `calc()` lub preprocesor typu SASS) i wielokrotność tej liczby przydzielić do odpowiednich klas jako szerokość kolumny. Założmy, że ja podzielę rodzica na 4 części, dla uproszczenia kodu:

```css
.grid-col--1 {
    width: 25%;
}

.grid-col--2 {
    width: 50%;
}

.grid-col--3 {
    width: 75%;
}

.grid-col--4 {
    width: 100%;
}
```

![Obrazek przedstawiający kolumny o różnych szerokościach]({{ site.url }}/assets/images/dlaczego-nie-potrzebujesz-bootstrapa/grid-4.png)

Kolumny posiadają teraz wymiary: 25% + 50% + 25% rodzica, czyli razem 100%. Jednak marginesy kolumn nie liczone są do ich szerokości. Na powyższym zdjęciu odstęp między kolumnami wykonany jest przy pomocy:

```css
.grid-col {
    box-sizing: border-box;
    border: 1px solid #fff;
}
```

`box-sizing` jest niezbędny, aby szerokość obramowania była brana pod uwagę przy obliczaniu szerokości. Możemy do stworzenia odstępu użyć paddingu wewnątrz kolumn lub marginesu na zewnątrz. Tutaj napotykamy na pierwszy problem z próbą stworzenia "generalnego schematu" czy "modułu" grida (problem, który nie występuje przy użyciu flexboxu): musimy indywidualnie (lub w pętli preprocesora) wyliczyć szerokości dla kolumn i aplikować marginesy w odpowiedni sposób, aby uzyskać jednakowe odstępy i szerokości kolumn dla dowolnego przypadku. Tutaj jest też przewaga rozwiązania skrojonego na miarę – jeśli to jest tylko kwestia napisania jednego "width", to po co używać do tego celu uniwersalnego systemu, który posiada szerokości kolumn, których być może nigdy nie użyjemy?

#### Breakpointy

Ostatnią rzeczą, którą musimy ustalić, jeśli tworzymy uniwersalny schemat, to stworzenie uniwersalnych breakpointów. Z definicji coś takiego jak uniwersalny breakpoint brzmi źle. Chodzi o opracowanie takich wymiarów strony, dla których treść zaczyna się źle prezentować. Tworzymy w tym miejscu breakpoint i aplikujemy nowe style.

Bootstrap wykorzystuje kilka breakpointów, nazwanych symbolicznie: `xs`, `sm`, `md`, `lg`, dla których możemy określić np, że nasza kolumna przestaje floatować i zajmuje 100% szerokości rodzica. Nic prostszego. Chcemy np. żeby kolumna typu `md` (md = 799px) zajmowała poniżej tego breakpointu szerokość 100%:

```css
@media (max-width: 799px) {
    .grid-col--md {
        width: 100%;
        float: none;
    }
}
```

I już. Teraz ponownie w pętli musielibyśmy wygenerować takie reguły dla wszystkich breakpointów i jesteśmy w przysłowiowym domu.

#### Grid: słowem zakończenia

Osobiście nie widzę sensu w tworzeniu lub stosowaniu modułu grida, jeśli możemy po prostu nazywać elementy według tego, czym są (BEM) i odpowiednie style aplikować bezpośrednio w CSS. Jak widać, te style nie są na tyle skomplikowane, żeby to rozwiązanie nie było opłacalne (gdzie zaoszczędzilibyśmy na pisaniu wielu tych samych reguł we wszystkich elementach-kolumnach). A nie jesteśmy wtedy ograniczeni przez sztywne breakpointy grida, które nie zawsze się sprawdzą.

### Modal

Okno modalne, dialog, to okno blokujące interakcję z pozostałą częścią strony, zazwyczaj poprzez przyciemnienie jej. Może to być powiększenie zdjęcia, potwierdzenie wykonania operacji lub inne.

Cały kod rozwiązania został opisany niżej.

```html
<div class="modal" id="sampleModal" role="dialog" aria-label="Podziękowanie za recenzję">
    <div class="modal_content">
        <button role="button" class="modal_close" aria-label="Zamknięcie okna dialogowego">Zamknij</button>
        <p>Dziękujemy za recenzję!</p>
    </div>
</div>
```

```css
.modal {
    position: fixed;
    top: 0; right: 0; bottom: 0; left: 0;
    z-index: 100;
    background: rgba(0,0,0,0.5);
    opacity: 0;
    pointer-events: none;
    transition: all 0.2s ease-in-out;
}

.modal--opened {
    opacity: 1;
    pointer-events: all;
}

.modal_content {
    position: absolute;
    top: 50%; left: 50%;
    transform: translate(-50%, -50%);
    background: #fff;
}

.modal_content {
    transform: translate(-50%, -50%) scale(0.8);
    transition: all 0.2s ease-in-out;
}

.modal--opened .modal_content {
    transform: translate(-50%, -50%) scale(1);
}
```

```js
(function modal () {
    function openModal (event) {
        const button = event.target;
        const id = button.getAttribute('data-modal');
        const modal = document.getElementById(id);
        modal.classList.add('modal--opened');
    }

    function closeModal (event) {
        event.target.closest('.modal').classList.remove('modal--opened');
    }

    const buttons = document.querySelectorAll('[data-modal]');
    for (let i = 0; i < buttons.length; i += 1) {
        buttons[i].addEventListener('click', openModal);
    }

    const closeButtons = document.getElementsByClassName('modal_close');
    for (let i = 0; i < closeButtons.length; i += 1) {
        closeButtons[i].addEventListener('click', closeModal);
    }
})();
```

#### Podstawowa struktura

W praktyce takie okno powinno zajmować cały ekran. Zacznijmy więc od tego:

```html
<div class="modal" id="sampleModal" role="dialog" aria-label="Podziękowanie za recenzję">
    <div class="modal_content">
        <button role="button" class="modal_close" aria-label="Zamknięcie okna dialogowego">Zamknij</button>
        <p>Dziękujemy za recenzję!</p>
    </div>
</div>
```

```css
.modal {
    position: fixed;
    top: 0; right: 0; bottom: 0; left: 0;
    z-index: 100;
    background: #fff;
}
```

Wyśrodkujmy treść modala:

```css
.modal_content {
    position: absolute;
    top: 50%; left: 50%;
    transform: translate(-50%, -50%);
    background: #fff;
}
```

Ustawiamy go wstępnie 50% od góry i 50% od lewej, a następnie przesuwamy o połowę własnej szerokości w obu kierunkach. Dzięki temu uzyskujemy idealne wyśrodkowanie.

Na tę chwilę efekt jest następujący:

![Obrazek przedstawiający otwarty modal z przyciemnionym tłem]({{ site.url }}/assets/images/dlaczego-nie-potrzebujesz-bootstrapa/modal-1.png)

#### Zmiana stanu modala

Zajmijmy się najpierw stanem, kiedy modal jest zamknięty (tzn. kiedy nie posiada klasy `.modal--open`). Taki modal nie może być widoczny, więc nadajmy mu `opacity: 0`. Szybko jednak zauważysz, że taki modal blokuje interakcję ze stroną. **On tam jest**, tylko go nie widać. Zasłania linki i przyciski swoim niewidzialnym ciałem. W związku z tym zamknięty modal powinien otrzymać również `pointer-events: none`, który załatwia sprawę tego, że blokuje interakcję.

Gdybyśmy ukrywali modal nie poprzez `opacity` tylko poprzez `display: none;` nie moglibyśmy animować jego otwierania.

Otwarty modal posiada przeciwieństwo tych stylów, tzn: `opacity: 1` oraz `pointer-events: all`. Dodajmy też `transition` aby zmiana stanu była animowana:

```css
.modal {
    position: fixed;
    top: 0; right: 0; bottom: 0; left: 0;
    z-index: 100;
    background: rgba(0,0,0,0.5);
    opacity: 0;
    pointer-events: none;
    transition: all 0.2s ease-in-out;
}

.modal--open {
    opacity: 1;
    pointer-events: all;
}
```

Ulepszmy animację, poprzez "wyłanianie się" faktycznego - białego - modala:

```css
.modal_content {
    transform: scale(0.8);
    transition: all 0.2s ease-in-out;
}

.modal--open .modal_content {
    transform: scale(1);
}
```

Wszystko na razie działa płynnie i bez bugów wizualnych (*nota bene* obecnych w modalu z BS).

#### Sterowanie modalem

Sterowanie modalem polega na dodaniu/usunięcie jego klasy `.modal--open`. Żeby nie być gołosłownym zrobię to w czystym JS. Zakładam, że moduł będzie wykorzystywał button `data-modal="[id modala]` do otwierania modala:

```html
<button role="button" data-modal="sampleModal">Otwórz sampleModal</button>
```

```js
(function modal () {
    function openModal (event) {
        const button = event.target;
        const id = button.getAttribute('data-modal');
        const modal = document.getElementById(id);
        modal.classList.add('modal--open');
    }

    function closeModal (event) {
        event.target.closest('.modal').classList.remove('modal--open');
    }

    const buttons = document.querySelectorAll('[data-modal]');
    for (let i = 0; i < buttons.length; i += 1) {
        buttons[i].addEventListener('click', openModal);
    }

    const closeButtons = document.getElementsByClassName('modal_close');
    for (let i = 0; i < closeButtons.length; i += 1) {
        closeButtons[i].addEventListener('click', closeModal);
    }
})();
```

Powyższy kod jest całkowicie elastyczny. Warto tylko zwrócić uwagę na zastosowany `Element.closest()` i [jego wsparcie](https://developer.mozilla.org/pl/docs/Web/API/Element/closest#Browser_compatibility).

#### Co można ulepszyć

1. Kod JS zakłada, że elementy istnieją w drzewie. Gdyby miało nastąpić ich stworzenie z poziomu JS, nic nie zadziała. W związku z tym polecam [delegować zdarzenia](https://davidwalsh.name/event-delegate).

2. Takie uruchomienie modala po pierwsze nie współpracuje z URLem (zmiana stanu strony, a czymś takim jest otworzenie modala, mogłoby zmieniać także hash, a zmiana hasha otwierać modal). Takie coś można rozwiązać poprzez zastosowanie [:target](http://www.kurshtml.edu.pl/css/etykieta,pseudoklasy.html): przycisk do otwierania byłby linkiem z odpowiednim hashem, modal miałby powiązane id, a link do zamykania cofałby do "#". W ten sposób możemy całkowicie zrezygnować z JS, jednak ograniczamy swoje możliwości (np. zamknięcie modala poprzez kliknięcie na jego ciemne tło).

3. Focus klawiatury nie targetuje modala tak jak trzeba. Po pierwsze focus działa na niewidzialnym modalu, po drugie otworzenie modala nie przenosi focusu do modala, a przy otwartym modalu nadal można focusować resztę strony.

4. [Dostępność okna modalnego](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_dialog_role)

Z punktami drugim, trzecim i czwartym ma problem nawet sam BS.

Na [swojej stronie](https://mortmortis.pl/portfolio.html) stworzyłem implementację rozwiązania z linkami (działa bez JS), a JS wykorzystuję do przeniesienia focusu jak trzeba (punkt 3.). Zamknięcie modala ustawia hash na to, co otworzyło tego modala. Nie jest to implementacja idealna (modale ogólnie są mało *dostępne*), ale działa dobrze. Mamy gotowe implementacje takich rozwiązań, które są dostępne: [a11y-dialog]( https://github.com/edenspiekermann/a11y-dialog).

## Podsumowanie

1. Staraj się **nie używać Bootstrapa**, bo naprawdę nie musisz go używać. Oszczędność czasu to tylko wymówka. Dbaj o jakość swojego produktu! Stosując Bootstrap trudno utrzymać jakość produktu!
2. Jeśli zdecydujesz się używać Bootstrapa pamiętaj, aby **dopasować paczki** [Bootstrapa](https://getbootstrap.com/docs/3.3/customize/) i [jQuery](http://projects.jga.me/jquery-builder/) do swoich wymagań!
3. Jeśli zdecydujesz się używać Bootstrapa, **nie unikaj stosowania [BEM](http://getbem.com/)**! Nie używaj klas z rodziny [Utilities](https://getbootstrap.com/docs/4.0/utilities/borders/). Staraj się **dużo kodu napisać samodzielnie** - wtedy zobaczysz, że BS nie daje Ci nic więcej niż twój własny kod.
4. Jeśli zdecydujesz się używać Bootstrapa zadbaj o to, aby wizualnie [Twoja strona się wyróżniała](http://adventurega.me/bootstrap/)!
5. **Nie polecaj nikomu Bootstrapa jeśli widzisz, że dopiero rozpoczyna naukę CSS**. Polecaj BS jako narzędzie do prototypowania stron i tylko tym, którzy użyją tego narzędzia jak trzeba.
6. **Minimalizuj liczbę tagów HTML** poprzez umiejętne używanie CSSa! Pamiętaj o [semantyczności](https://tutorials.comandeer.pl/html5-blog.html).

Mam nadzieję, że ten post przynajmniej trochę zachęci programistów do rezygnacji z Bootstrapa, intensywniejszej nauki CSSa i w miarę możliwości zastępowania kodu uniwersalnego, kodem skrojonym na miarę - kodem lżejszym, czytelniejszym, własnym i co najwazniejsze: kodem, który przyczynił się do naszego rozwoju.

Post będzie prawdopodobnie rozbudowywany o kolejne komponenty. Propozycje proszę zgłaszać w komentarzach.

**Podziękowania**: [Albert](https://wolszon.me), [Comandeer](https://comandeer.pl)

