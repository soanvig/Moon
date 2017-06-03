---
layout: post
title: Na czym polega programowanie?
comments: true
excerpt: Próba podjęcia tematu opisania programowania osobom, które nie robią tego od kilku lat.
---

## O czym będzie ten wpis?

Wpis ten będzie próbą podjęcia tematu opisania programowania osobom, które nie robią tego od kilku lat. Nie ma znaczenia doświadczenie w programowaniu, w gruncie rzeczy nieważny jest język programowania.

Pragnę również podkreślić, że nie jest to praca w żadnym razie naukowa ani wymagająca specjalistycznej wiedzy. Postaram się, aby lektura była lekka, choć moje pióro do lekkich nie należy :-)

## Definicja programowania

![]({{ site.url }}/assets/images/na-czym-polega-programowanie/programming.jpg)

Na początku warto jest zapomnieć o wszystkim czego uczą o programowaniu w szkołach. W praktyce uczą tego osoby z nikłym doświadczeniem i brakiem pasji do tej dziedziny wiedzy.

Według mnie

> Programowaniem można nazwać każdą czynność, której celem jest znalezienie rozwiązania problemu, a następnie przedstawienie go [rozwiązania] w postaci uporządkowanych kroków: algorytmu.

*Definicja* ta nieco odbiega od powszechnego wyobrażenia programowania. 

### Czym powszechnie nazywamy programowanie?

![]({{ site.url }}/assets/images/na-czym-polega-programowanie/chinese.jpg)

Co programujemy? Pralkę, komputer (w domyśle: programy komputerowe)... Pewnie jeszcze parę innych urządzeń, których nazw nie potrafię sobie przypomnieć. Jak już wspomniałem odbiega to od mojej definicji programowania. Programowanie pralki (jak i komputera) polega na wydawaniu poleceń. Całkiem prosta czynność, nie wymagająca zdolności przywódczych, gdyż urządzenia są całkowicie uległe (a przy tym całkowicie głuche!).

Ja tymczasem przedstawiam programowanie jako proces myślowy, rozważania, pewną abstrakcję.

(podkreślam, że w dalszej części wpisu będę używał słowa "programowanie" w definicji podanej powyżej, chyba że inaczej wynika z kontekstu)

### *Kodzenie*

Jak zatem nazwać proces pisania kodu (wydawania poleceń)? Posłużę się nazwą "kodzenie" (z ang. *coding*). "Kodzę" jest całkiem niezłym (m.in. łatwym do odmieniania) zamiennikiem nieco dłuższej i niewygodnej formy "piszę kod".

## Tyle tekstu przeczytałem, a nadal nie wiem, na czym polega to programowanie!

No dobrze, postaram się to wyjaśnić na przykładzie.

### Przykład

Od czegoś cały proces programowania trzeba zacząć. Pamiętajmy, że programowanie polega na rozwiązaniu problemu (zadania). Zarzucę więc problemem życiowym, nieco sztampowym, nieco satyrycznym:

![]({{ site.url }}/assets/images/na-czym-polega-programowanie/scrambled-eggs.jpg)

> Zrób jajecznicę z kiełbasą i cebulką.

Wydaje się proste, prawda? Jestem pewien, że na pytanie *"W jaki sposób zrobisz jajecznicę?"*, dużo osób powiedziałoby *"Po prostu ją zrobię"*, ponieważ zrobienie jajecznicy jest czynnością prostą i logiczną. Oczywiście na *just because* można zakończyć temat, spróbuję jednak rozbić go na części pierwsze.

#### Cel

Najpierw znajdźmy cel zadania. To zdecydowanie pierwsza czynność, którą należy wykonać podczas szukania rozwiązania problemu. Cel musi być jasno zdefiniowany, ponieważ musimy wiedzieć, do czego tak naprawdę dążymy.
Im cel będzie bardziej ogólnie określony **tym łatwiej** będzie rozwiązać problem.

Wbrew pozorom celem zadania *Zrób jajecznicę z kiełbasą i cebulką* nie jest 'jajecznica z kiełbasą i cebulką', tylko 'jajecznica'. Jak wspomniałem - staramy się ugólnić problem. Dopiero w kolejnych krokach dodane zostaną *modyfikatory* algorytmu (kiełbasa, cebulka), dzięki którym osiągniemy końcowy cel.

**Dla dociekliwych:** na dobrą sprawę można cel skrócić nawet do 'zrób', jednak dla 'zrób' nie jesteśmy w stanie ułożyć żadnego algorytmu i stąd wiemy, że takie uogólnienie jest przesadzone.

Im bardziej złożone zadanie, tym bardziej ogólny cel należy zdefiniować, a następnie po kolei dodawać do niego modyfikatory. To wręcz fundamentalna zasada, której należy przestrzegać przy skomplikowanych logicznie układach.

A więc algorytm do osiągnięcia celu może wyglądać tak:

1. Wbij jajka na patelnię.
2. Smaż aż białko i żółtko się zetną.
3. Podaj.

Z pewnością w chwili 'podania' mamy jajecznicę przy wykonaniu miminalnej i wystarczającej liczby kroków.

Można teraz się zastanowić nad kilkoma kwestiami i rozszerzyć algorytm osiągnięcia celu o kwestie organizacyjne...

1. Skąd weźmiemy jajka? (odp. z lodówki, a jeśli ich tam nie ma, to kupimy)
2. Skąd będziemy wzięli, że jajka są odpowiednio ścięte? (odp. musimy znać prefencje konsumentów)
3. W jaki sposób podamy? (odp. na talerzu odpowiednio dużym (małym))

... ale przejdźmy do modyfikatorów.

#### Modyfikatory

Nasz "program" działa, ponieważ otrzymaliśmy jajecznicę. Nie spełnia jednak wszystkich wymogów (nie *"zaimplementowaliśmy"* kiełbasy i cebulki).

Trzonem programu jest algorytm, dzięki któremu można osiągnąć cel. Mając to utrzymujemy porządek w kodzie i możemy zacząć dodawać modyfikatory. Może się czasem okazać, że algorytm bez modyfikatorów nie ma racji bytu. I tutaj pomaga doświadczenie w programowaniu - pomaga dobrać dla trzonu programu minimalną i wystarczającą liczbę modyfikatorów.

O modyfikatorach można myśleć jak o modułach dodających nowe *funkcje* (naciągnijmy kiełbasę do funkcji "smaku kiełbasy"). Niech zatem kiełbasa (i analogicznie cebulka) będzie takim modułem. Moduł oczywiście istnieje (został przez kogoś innego stworzony) i leży w lodówce. Trzeba tylko po niego sięgnąć, przygotować go i w odpowiedni sposób dodać do trzonu.

Kiełbasa w związku z tym posiada własny algorytm-podprogram: przygotowanie. Należy ją przynajmniej pokroić w kostkę. Dodać ją można prosto do jajek (najpierw dodamy jajka, potem kiełbasę), lub przesmażyć wcześniej (najpierw dodamy kiełbasę, potem jajka). Sposób dodania modułu do trzonu programu zmienia nam więc końcowy efekt - smak (na szczęście wymagania w tej kwestii nie są określone, więc możemy nie solić).

#### Podsumowanie przykładu

Mówienie o jajecznicy jako "celu" i o kiełbasie jako "module" brzmi zabawnie i sam się z tego śmieję. Jednak umiejętność szybkiej identyfikacji obiektów i ich funkcji jest kluczowa w programowaniu. Dzięki temu wiemy, co jest celem, a co modyfikatorem celu i nie gubimy się we własnym algorytmie, a co najważniejsze - jesteśmy w stanie go w ogóle stworzyć.

Z doświadczenia wiem, że bardzo kusi pisanie od początku wszystkiego wraz z modyfikatorami. Tylko że taki sposób często kończy się niedziałającym programem (a już na pewno bałaganem, który utrudnia znalezienie ewentualnych błędów). Przypomina to budowanie wieżowca. Zamiast po kolei budować: fundamenty, dźwigary, podłogi, ściany działowe, rury i tak dalej; buduje się kompletne pierwsze piętro (wraz z pomalowaniem ścian), a dalej wznosi kolejne piętra, ale zawsze kompletne przed wzniesieniem następnego. Cytując klasyka:

> Andrzej, to jeb***.

## Nauka programowania

![]({{ site.url }}/assets/images/na-czym-polega-programowanie/book.jpg)

### Szkoła

Próba nauczenia programowania dzieci czy starszych uczniów poprzez przepisywanie kodu, czy nawet jakieś kulawe samodzielne pisanie programów jest wysoce niewydajne.

Po latach programowania mogę powiedzieć, że język programowania nie ma znaczenia. Mogę przeskakiwać między językami i rozwiązywać problemy bez większego trudu. Wynika to z tego, że nauczyłem się faktycznie programować.

Tymczasem **każdy** na początku swojej programistycznej kariery zmaga się z problemami natury składniowej języka (błędy podczas uruchamiania programu wynikające z nieprawidłowego zapisu kodu). Jest to całkiem naturalne, ponieważ język programowania zazwyczaj **daleki jest** od podobieństwa do języków ludzkich, mimo usilnych prób ich porównywania.

Zamiast skupieniu się na rozwiązaniu zagadnienia (zadania) początkujący programista traci czas i nerwy na błędy składniowe. Kończy się to frustracją i porzuceniem tematu. I powtarzam: **jest to całkowicie normalne**.

Oczywiście, od czegoś trzeba zacząć. Sam zaczynałem od razu od kodzenia. Teraz piszę z perspektywy czasu. Według mnie najlepiej byłoby prowadzić naukę równolegle: najpierw przygotować rozwiązanie na brudno na kartce, następnie je spróbować zaprogramować.

### Nauka samodzielna

Prawda jest taka, że jeśli ktoś nie zainteresuje się programowaniem i samodzielnie mu nie poświęci czasu w domu, to w szkole (na studiach) niczego poważnego się nie nauczy. Trzeba temu poświęcić czas. Dużo czasu. Tego nie da się wkuć. To trzeba poczuć. To przychodzi samo, stopniowo i z czasem. Nigdy nie można powiedzieć, że się jest w tym doskonałym. Zawsze można być lepszym.

**Programowanie to myślenie** - ni mniej, ni więcej. Specyficzny sposób myślenia, to prawda. Co ciekawe, ten sposób przekłada się potem na codziennie myślenie nie tylko podczas programowania.

## Programowanie a język programowania

![]({{ site.url }}/assets/images/na-czym-polega-programowanie/tools.jpg)

**Język programowania nie ma znaczenia**. To jest rzecz, której staroszkolni nauczyciele najczęściej nie rozumieją i starają się forsować wyższość jednego języka nad innym.

**Język programowania to narzędzie**. Mogę zmienić je na inne, o nieco innych właściwościach (wadach i zaletach). Mogę z narzędzia efektywnego i trudnego w obsłudze przejść na łatwe, lecz mniej wydajne. Każdy język posiada swoje wady i zalety. Gdy chodzi o połączenie dwóch desek ze sobą mam do wyboru: użycie gwoździ i młotków, wkrętów i śrubokręta, klejów, na upartego nawet zszywacza tapicerskiego. Dokładnie tak samo jest z językami programowania: każdy ma swoje zastosowanie (wady i zalety), ale przy pomocy każdego można uzyskać satysfkacjonujący końcowy efekt.

Język programowania to narzędzie wykorzystywane przy procesie wdrażania **wymyślonego** algorytmu. Najważniejsze jest **myślenie**, nie język. Pierwsze co zrób, gdy uczysz się programować, to zapomnij o języku programowania i *zastanów się*, co **tak naprawdę** ma się dziać.

## Programowanie a inne dziedziny

Choć na co dzień programuję na komputerze, to samo myślenie pomaga mi na studiach przy rozwiązywaniu problemów matematycznych czy innych - stworzenie działającego w określony sposób układu elektropneumatycznego to doskonały przykład programowania i algorytmiki. Pierwsze co robię, to określam sobie jasno cel i realizuję go, a dopiero po tym dodaję **po kolei** modyfikatory, które zapewnią mi działanie układu zgodnie z wymaganiami zadania.

Zdecydowanie programowanie jest powiązane przede wszystkim z pisaniem kodu. W praktyce jednak można je odnieść do zadań z różnych dziedzin życia. Czasem lepszym rozwiązaniem jest rozbicie zadania na części pierwsze i wykonanie go krok po kroku, niż naturalna próba ogarnięcia wszystkiego naraz.

## Podsumowanie

> - Na czym polega programowanie?
> - Na znalezieniu prostej drogi do rozwiązania.

Nie poruszyłem tutaj wielu kwestii (utrzymanie kodu, szukanie błędów), ale to dlatego, że według mnie są bardziej związane z kodzeniem niż programowaniem *per se*.

Gdybym miał więc podać jak najkrótszą definicję programowania, powiedziałbym:

> Programowanie - po kolei.

...co dla człowieka jest wyjątkowo nienaturalne.

Dziękuję za uwagę,

Mateusz Koteja (*Soanvig*)



