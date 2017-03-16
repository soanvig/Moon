---
layout: post
title: IEIE Core - podstawowe narzędzia
tags: [DSP2017, IEIE, Ruby]
comments: true
excerpt: Stworzenie podstawowych funkcji i klas wykorzystywanych przy pracy z IEIE Core.
---

## Wstęp

Dla przypomnienia: wpis ten dotyczy projektu **Infinity Engine Incredible Editor**, części *Core* aplikacji (zob. wpis ["Daj się poznać"](/daj-sie-poznac)).

Aby móc efektywnie tworzyć nowe moduły obsługujące kolejne formaty plików należy wyodrębnić elementy powtarzalne, takie same dla każdego formatu pliku. Spróbuję to w tym wpisie zrobić.

Należy to zrobić raz, a dobrze, ponieważ będą to najczęściej wykorzystywane elementy kodu w tym projekcie.

## Wzorzec

Korzystam oczywiście z [bazy danych Gibberlings3](http://gibberlings3.net/iesdp/file_formats/ie_formats/) w celu identyfikacji budowy plików binarnych, w których przez Infinity Engine przechowywane są dane.

Każdy plik posiada swój - jak to nazwałem - wzorzec (ang. *pattern*). Wzorzec jest tabelą posiadającą swoją nazwę (np. *Header*), oraz wiersze, gdzie każdy wiersz stanowi *pole* i posiada: *nazwę*, *wielkość* i *offset*. 

- Nazwa jest stosowana jako czytelny dla człowieka identyfikator.
- Offset jest informacją o położeniu pola w pliku binarnym
- Wielkość jest informacją o ilości bajtów przynależących do pola.
- Pole w pliku binarnym jest *wartością* samo w sobie.

Format plików może oczywiście zawierać kilka wzorców (każdy inny). Można więc stworzyć ogólny szablon (klasę) wzorca (oraz funkcje wykorzystywane dla każdego z nich), a następnie napisać dla każdego formatu plików klasy specyficzne dla danego formatu, dziedziczące po klasie ogólnej. Brzmi to nieco enigmatycznie, więc pokażę na przykładzie.

### Przykład

Niech klasa ogólna (tabela, jednostka podstawowa każdego formatu plików) nazywa się *Block*, a wzorzec: *pattern*. Zacznę od przygotowania przykładowego wzorca.

#### Wzorzec

Wzorzec musi zawierać informację o: nazwie pola, aby zapisać je w formie czytelnej dla człowieka (*name*); offsecie pola (*offset*); oraz o rozmiarze pola (*size*).

Nazwa niech odpowiada opisowi (nazwie) pola podanego na stronie Gibberlings3, offset niech będzie ciągiem znaków reprezentującym liczbę w formacie szesnastkowym, a rozmiar niech będzie liczbą.

Dlaczego offset ma być ciągiem znaków szesnastkowym, a rozmiar liczbą?

Rozmiar zazwyczaj zawiera się w przedziale 0-20 (większych pól raczej się nie przewiduje - nie ma potrzeby ich stosowania), natomiast offset teoretycznie może osiągać duże wartości. Przepisując wartości z tabeli do skryptu trzeba by było siedzieć z kalkulatorem w dłoni (lub konwerterem systemów liczbowych). I po co? Dlatego przepiszę offset tak jak jest podany w tabeli, a rozmiar szybko przeliczę w głowie.

Spróbuję napisać krótki wzorzec dla [nagłówka formatu .BIF](http://gibberlings3.net/iesdp/file_formats/ie_formats/bif_v1.htm#bif_v1_Header)

```ruby
pattern = {
    signature: [4, '0000'],
    version: [4, '0004'],
    file_count: [4, '0008'],
    tileset_count: [4, '000c'],
    file_offset: [4, '0010']
}
```

Powyższa zmienna `pattern` jest typu `Hash`, gdzie kluczem jest nazwa pola, a wartością tablica: `[size, offset]`.

#### Blok

Blok musi być klasą, która będzie przyjmowała jako parametry: *źródło* (tablicę bajtów [zob. wpis ["IEIE - format zapisu plików binarnych"](/ieie-format-zapisu-plikow)]), *offset startowy* (numer bajtu w źródle, od którego zaczyna się blok - w jednym źródle znajdziemy kilka bloków) oraz *wzorzec* (hash z poprzedniej sekcji).

```ruby
class Block
    # :values - wartości ze wzorca przeparsowane przez Block
    # :start - offset, od którego zaczyna się blok w źródle
    # :end - offset, na którym kończy się blok (wykorzystywane przede wszystkim przy seryjnym parsowaniu bloków)
    attr_reader :values,
                :start,
                :end

    def initialize( source, start, pattern )
        @values = {}
        recreate( source, start, pattern )
    end
    
    # Dzięki tej metodzie będziemy mogli odwoływać się do wartości poprzez blok[:pole] zamiast blok.values[:pole]
    def []( name )
        @values[name]
    end

    private

    def recreate( source, start, pattern )
        @start = start
        pattern.each do |name, field|
            # Konwertowanie na liczbę
            field[0] = field[0].to_i(16) if field[0].is_a? String
            offset = start + field[0]

            # Wybranie ze źródła field[1]-ilości bajtów, zaczynając od indeksu równemu offsetowi
            result = source[ offset, field[1] ]

            # Przesunięcie znacznika końca bloku o rozmiar nowego pola
            @end = offset + field[1]

            # Zapisanie do zmiennej
            @values[ name.to_sym ] = result
        end
    end
end
```

Teraz jestem w stanie dziedziczyć po tej klasie, i tak dla przykładu tabeli *Header* BIF:

```ruby
class BIFF::Header < Block
    @@pattern = {
        signature: [4, '0000'],
        version: [4, '0004'],
        file_count: [4, '0008'],
        tileset_count: [4, '000c'],
        file_offset: [4, '0010']
    }
    
    def initialize( source, start )
        super( source, start, @@pattern )
    end
end
```

I tyle. Po wywołaniu nowej instancji klasy `BIFF::Header` nasz blok zostanie załadowany. Teraz możemy do niego (lub do `Block`) dopisywać nowe metody. W przyszłości w `Block` pojawi się metoda przygotowująca blok do zapisania - czyli zamieniająca istniejące już w bloku dane na tablicę bajtów, używając tego samego wzorca (ale w sposób *odwrotny*).

## Konwertery

Na powyższym skończyłem pierwszą wersję programu **Core**. Następnym krokiem było przygotowanie konwerterów, aby zamienić bajtową wersję danych na formę czytelną dla człowieka.

Niestety przy tablicach nie jest podany typ danych jaki się tam znajduje (liczba, słowo?), jest tylko typ samego pola, a to nie to samo. Tak więc metodą prób i błędów, a także metodą "na logikę", udało mi się zidentyfikować kilka typów danych:

- słowo (każdy bajt to liczba, każda liczba to litera w kodzie ASCII),
- tablica (każdy bajt to tablica 8-miu bitów, każdy bit to prawda [1]/fałsz [0]),
- liczba (zestaw bajtów to liczba zapisana w formacie little-endian - normalnie ostatni bajt liczby jest pierwszym, przedostatni drugim i tak dalej),
- boolean (zestaw zerowych bajtów to fałsz, każdy inny to prawda).

I dla tych typów przygotowałem konwertery. Wszystkie konwertery operują na tablicy bajtów, w związku z czym rozszerzyłem klasę `Array` metodą `.convert_to( type )`. Sama konwersja jest prosta, dlatego nie będę tutaj umieszczał jej kodu.

Typ danego pola umieściłem jako trzecią wartość tablicy wzorca, w przypadku nieznanego typu zostawiam jako wartość pola wyjściową wartość - tablicę bajtów, natomiast samo konwertowanie odbywa się w `Block.recreate`

## Podsumowanie

Stworzenie podstawowej i uniwersalnej jednostki każdego formatu pliku (klasa `Block`) było szczególnie ważne dla rozwoju całego projektu, ponieważ klasę tę wykorzystywałem nagminnie. Konwerter jest używany przede wszystkim podczas parsowania bloku.

### Co można poprawić w tym kodzie? 

1. Przede wszystkim wzorzec bloku umieszczamy w klasach odpowiadajacych tabelom danego formatu. Lepszym pomysłem jest przerzucenie tych wzorców do osobnych tabel, zapisanych np. w formacie [YAML](http://yaml.org/) (zob. parser tabel <https://github.com/IE-IE/core/blob/master/lib/tables.rb> oraz przykładową tabelę: <https://github.com/IE-IE/core/blob/master/lib/tables/biff.yml>).
2. Warto przypisać do pola wartość domyślną, którą wykorzystywać będę w przyszłości podczas przygotowywania bloku do zapisu. Być może użytkownik nie przypisze polu żadnej wartości - w takim układzie należy wykonać fallback do wartości domyślnej, aby zachować ciągłość pliku binarnego.
3. Ciąg znaków może zawierać znaki zakodowane w innych kodowaniach niż podstawowy ASCII. Należy przygotować funkcję enkodującą ciąg znaków na odpowiedni format (tutaj pojawiły się problemy, a omówione zostaną w przyszłym wpisie :-) ).
4. Aby umożliwić zapisywanie bloku należy przygotować konwertery zamieniające odpowiednie typy na tablice bajtów.
