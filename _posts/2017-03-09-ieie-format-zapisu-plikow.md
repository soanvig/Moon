---
layout: post
title: IEIE - format zapisu plików binarnych
tags: [DSP2017, IEIE, Ruby, binarny]
comments: true
excerpt: Analiza budowy plików binarnych, w których przechowywane są dane przez silnik Infinity Engine
---

## Wstęp - przypomnienie, co to jest Infinity Engine

Silnik (gier komputerowych) Infinity Engine pozwala na tworzenie gier cRPG. Napisany został przez BioWare i wykorzystywany przede wszystkim przez studio Black Isle.

Gry jakie w nim powstały z pewnością są wszystkim dobrze znane, bowiem uchodzą za legendy komputerowych gier RPG:

- *Baldur’s Gate*
- *Icewind Dale*
- *Planescape Torment*

(nie wymieniam dodatków, rozszerzeń czy kolejnych części tego samego tytułu).

Jeśli gra jest dobra, z pewnością przegramy w nią co najmniej kilkadziesiąt godzin, nieraz przechodząc ją po kilka razy. Z czasem jednak – co nieuniknione – zacznie doskwierać nam nuda. Wtedy z pomocą przychodzą, a jakże, modyfikacje!

## Tworzenie modu

Tylko czy zastanawialiśmy się, w jaki sposób mody zostały stworzone? Spróbujmy stworzyć modyfikację pliku przedmiotu (stwórzmy po prostu nowy przedmiot) dla gry *Baldur’s Gate* (część pierwsza).

Z pewnością punktem wyjścia modu dla nieudokumentowanego silnika gry (ponieważ oficjalna dokumentacja Infinity Engine nie jest dostępna) są pliki oryginalne, stworzone przez twórców.

Dobrze, znajdźmy zatem jakiś plik przedmiotu, żeby móc go skopiować, edytować i ostatecznie dodać na nowo do gry (i dać naszej postaci za pomocą kodu).

A właściwie gdzie jest plik przedmiotu? Najbliżej będzie plik data/ITEMS.BIF.
Czyżby tablica plików? Nic bardziej mylnego! A w każdym razie: na etapie naszej obecnej wiedzy.

## Analiza pliku ITEMS.BIF

Pierwsze (jak i kolejne) linijki pliku otworzonego w Sublime Text wyglądają tak:

```
4249 4646 5631 2020 0305 0000 0000 0000
9c96 0700 0000 0000 4954 4d20 5631 2020
6437 0000 6637 0000 0000 00a4 1424 0004
2000 0000 0000 0000 0000 2020 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000
0100 0000 00a4 1424 0004 0000 0000 00a4
1424 0004 0000 0000 6537 0000 6737 0000
```

i jest to 32365 linijek łącznie.

Ale przecież w jakiś sposób ktoś stworzył edytory/przeglądarki plików (np. NearInfinity). Czyżby informacja była w jakiś sposób zakodowana?

Otóż tak! Okazuje się, że jest to plik zapisany w systemie liczbowym szesnastkowym (na co dzień używamy systemu liczbowego dziesiętnego).

Jeden bajt w systemie szesnastkowym to 2 znaki (czyli np. pierwsze 4 znaki 4249 to dwa bajty: 42 i 49). Zapis bajtów parami jest stosowany jedynie dla czytelności.

## Pierwszy kod

Pora napisać pierwszy kod. Wybrałem język Ruby (ponieważ jest to mój ulubiony język programowania).

Na pewno przyda się podzielić cały plik na pojedyncze bajty tak, aby potem móc na nich wygodnie pracować. Chcielibyśmy zachować zapis w formacie szesnastkowym, ponieważ jest on dla nas na tym etapie wystarczająco czytelny.

Niestety, nie można po prostu otworzyć takiego pliku (binarnego) i pociąć całej treści na pary znaków. Skończymy wtedy bowiem z informacją zapisaną w postaci stringów: \x2050 – i bynajmniej nie jest to szczególnie wygodna forma do pracy.

Można jednak otworzyć plik jako plik binarny, a następnie pobierać pojedyncze bajty. Aby jako tablicę wynikową dostać taką, jak podana poniżej, należy bajty przekonwertować na liczbę (`.to_i`), a następnie liczbę na ciąg znaków w formacie szesnastkowym (`to._s(16)`).

Fragment tablicy bajtów, jaką chcemy osiągnąć:

```ruby
['42', '49', '46', '46', '56', '31', '20', '20', ...]
```

Kod zamieniający wszystkie bajty w pliku na tablicę bajtów podpiąłem od razu pod klasę File jako metodę klasową. Dodałem także komentarze, aby osoby słabo znające Ruby się połapały.

```ruby
class File
  def self.get_bytes( location )
    bytes = [] # Przygotuj tablicę, w której zapiszemy bajty

    # Otwórz plik jako tylko-do-odczytu i binarny
    # Dla każdej linijki pliku wykonaj blok
    self.open( location, 'rb' ) do |f|
      # Dla każdego pliku w linijce wykonaj blok
      f.each_byte do |b|
        # Zamień bajt na liczbę i na ciąg znaków w formacie .16
        b = b.to_i.to_s(16)
        # W wyniku konwersji traci się informację o tym,
        # że bajt powinien mieć 2 znaki (dla czytelności)
        # Dodaj utracone zero na początku
        b = '0' + b if b.length == 1
        # Dodaj nowy bajt to tablicy bajtów
        bytes << b
      end 
    end
    
    # Zwróć tablicę bajtów
    bytes
  end
end

# Przykład użycia:
file ='data/ITEMS.BIF'
bytes = File.get_bytes( file )
bytes # => ['42', '49', '46', '46', ...]
```

W tej chwili posiadamy dokładnie to, co widzieliśmy na ekranie edytora tekstu, a każdy bajt zapisany jest w osobnej komórce w tabeli.

Co dalej?

## Wykorzystujemy pracę innych ludzi

Cóż, same bajty niewielką nam dają informację o zawartości pliku, choć gdybyśmy je wszystkie przekonwertowali na litery ASCII (każda litera to bowiem liczba w formacie ASCII, a nasze bajty to przecież liczby), to co nieco można odczytać, np. „BOW01” (identyfikator jednego z łuków w grze). Jesteśmy więc na dobrym tropie.

Jeśli odpowiednio przeszukamy sieć, to znajdziemy wszystko co chcemy. Okazuje się, że publicznie dostępna jest baza analiz plików Infinity Engine stworzona przez programistów na przestrzeni lat. Jak tego dokonali - domyślam się, ale to i tak tytaniczna praca.

http://gibberlings3.net/iesdp/file_formats/ie_formats/

Znajdźmy format .biff (przypominam, że analizujemy plik ITEMS.bif):

http://gibberlings3.net/iesdp/file_formats/ie_formats/bif_v1.htm

Jak widać cały plik podzielony jest na trzy główne sekcje (Header, File Entries oraz Tileset Entried).

Każda sekcja podzielona jest na kolejne podsekcje, z których każda posiada offset, size oraz size type.

- **offset** - zapisany w postaci szesnastkowej; oznacza, od którego bajtu w pliku rozpoczyna się dana podsekcja.
- **size** - zapisany w postaci szesnastkowej; oznacza liczbę bajtów należącą do danej podsekcji (rozpoczynając od bajtu na pozycji **offset**).
- **size type** - ta informacja nie jest w naszym programie potrzebna.

Nagłówek (Header) naszego pliku posiada:

- Signature
- Version
- Count of file entries
- Count of tileset entries
- Offset to file entries

Pierwsze, co zrobiłem, to przekonwertowałem przykładowy offset na liczbę w systemie dziesiętnym:

```ruby
(offset podsekcji Version)
'0004'.to_i(16) 
# => 4
```

W edytorze odliczam sobie ten offset i kopiuję 4 następne bajty (ponieważ size = 4): `5631 2020` i przy wykorzystaniu metody `Array#pack` zamieniam te bajty na tekst ASCII:

```ruby
['56312020'].pack('H*')
# => "V1  "
```

To już coś. Otrzymaliśmy dokładnie to, co podane jest na stronie Gibberlings3. Możemy więc założyć, że jesteśmy na dobrej drodze.

## Co dalej?

Zrozumieliśmy, technikę leżącą u podstaw zapisu plików w Infinity Engine.
Nauczyliśmy się wgrywać te pliki do pamięci programu i zapisywać je w wygodnej formie (formie bezpośrednio związanej z oryginałem i nie fałszującej danych – co jest bardzo ważne, jak się przekonamy).

Znaleźliśmy bazę wiedzy, dzięki której będziemy w stanie odszyfrować i uporządkować dane w każdym z formatów plików.

Kolejnym krokiem (oczywiście w kolejnym wpisie) będzie zamiana danych w pliku na dane uporządkowane według wzorca z naszej bazy wiedzy. Stworzę w tym celu coś, co nazywam *jednostką podstawową* - klasę `Block`.

Przypominam, że kod projektu znajduje się pod adresem:
https://github.com/soanvig/ie-ie 
a omawiany kod znajduje się w: `lib/helpers.rb`.

Dziękuję za uwagę i życzę miłego dnia :-)
