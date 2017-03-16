---
layout: post
title: Tworzenie odpowiednika Ruby on Rails
tags: [DSP2017, IEIE, Ruby, Ruby on Rails]
comments: true
excerpt: Próba stworzenia podstawowego frameworku działającego jako API i mającego podobny sposób działania jak Ruby on Rails
---

## Wstęp

Oprócz bibliotek obsługujących pliki silnika Infinity Engine, **Infinity Engine Incredible Editor** w założeniu miał posiadać także serwer API, umożliwiający komunikację z bibliotekami poprzez HTTP.

Teraz za API robi *Ruby on Rails* (w wersji API), ale nie zawsze tak było.
Kiedyś nie chciałem czegoś tak "ciężkiego" jak cały wielki i potężny RoR. Niemniej podobała mi się zasada działania RoR, no i byłem do niego przyzwyczajony.

Postanowiłem więc stworzyć odpowiednik Rails. Założenia projektowe wyglądały następująco:

1. Program powinien posiadać możliwość stworzenia routingu w sposób identyczny, jak w Rails `'/api/chitin/filetypes' => 'API#chitin'` (gdzie po prawej stronie `API`to nazwa kontrolera, a `chitin` to nazwa metody).
2. Router powinien wspierać zmienne w adresach i konwertować je na `params[:nazwa_zmiennej]`, np. `'/api/chitin/files/:type' => 'API#chitin'` (zmienna `:type` oznaczona dwukropkiem).
3. Klasy powinny być wczytywane dynamicznie (tzn. przy każdym wywołaniu, klasa jest na nowo ładowana), aby w trybie deweloperskim nie restartować serwera po każdej zmianie w kodzie.
4. Program powinien renderować strony przy użyciu ERB (*instance variables* powinny być dostępne wewnątrz widoku), a widok powinien być odseparowany od logiki kodu zawartej w kontrolerze.
5. Aplikacja powinna odpowiadać w JSONie (oznacza to, że widoki są *de facto* metodą zamiany zmiennych na odpowiedni wzór JSON).

Przeanalizuję teraz poszczególne punkty.

## 1. Routing podstawowy

Stworzenie podstawowego routingu rozpocząłem od stworzenia klasy `Route`, w której w zmiennej klasowej `@@routes` zapisane są ścieżki routingu:

```ruby
class Route
  @@routes = {
    'GET' => {
      '/api/chitin/filetypes' => 'API#chitin',
      '/api/chitin/files/:type' => 'API#chitin', # Sposób interpretacji zmiennej jest przedstawiony w punkcie 2. wpisu.
      '/api/chitin/full' => 'API#chitin_full',
      '/api/chitin/set' => 'API#chitin_set' # ?location=...
    },
    'POST' => {

    }
  }
end
```

Zakładam, że każda jedna interpretacja ścieżki (każdy request) będzie instancją klasy Route. Potrzebuję więc metody `initialize` (konstruktor):

```ruby
def initialize( request )
    target = @@routes[request.request_method][request.path].split('#')

    @class  = target[0]
    @method = target[1]
end
```

Zmienna `request` to zmienna otrzymana od serwera WEBrick - parametr `.request_method` zawiera informację o typie żądania (POST, GET) natomiast `.path` o żądanej ścieżce. Ostatecznie rozbijam znaleziony route na znaku `#` i przyjmuję, że żądanie powiniem przekierować do klasy znajdującej się po lewej stronie krzyżyka, do metody z prawej strony.

I w tym momencie router kończy zadanie, a kontroler całego "frameworku" zajmie się resztą.

## 2. Parametry routingu (w tym zmienne)

Aby móc interpretować część tekstu jako zmienną, należy się zastanowić nad przyjęciem wygodnej metody zrobienia tego. Początkowo strasznie kombinowałem, kiedy spróbowałem w końcu z wyrażeniami regularnymi (regexp - *regular expression*). Dowiedziałem, że istnieje coś takiego jak *named regular expression*. Innymi słowy, kiedy silnik zintepretuje wyrażenie regularne i porówna je z zadanym ciągiem, wypluje na swoim wyjściu nie tablicę z "dopasowaniami", tylko tablicę ze zmiennymi - każda będzie miała swoją nazwę zdefiniowaną w wyrażeniu regularnym, a wartość odpowiadającą wartości z porównywanego wzorca.

Na przykładzie: posiadam wzorzec regexp: `/user/get/?<id>` i ciąg do porównania: `/user/get/5`. Po wykonaniu i porównaniu wyników otrzymam zmienną `id = 5`.

Dobrze, ale założyłem, że zmienne w ścieżkach routera podaję przez dwukropek. Cóż, w takim razie nie pozostaje nic innego niż przepuszczenie tych ścieżek przez funkcję zmieniającą je na formę regexa podaną powyżej:

```ruby
def route_to_regexp( route )
    new_route = []

    # podziel ścieżkę na części według '/' 
    route.split('/').each do |el| 
      # jeśli fragment ścieżki zaczyna się od : (jest zmienną) - przetraw go!
      if( el.to_s[0] == ':' )
        # zapisz nazwę zmiennej jako ciąg znaków od drugiego znaku (1), do ostatniego (-1) - w ten sposób usuniemy dwukropek
        name = el[1..-1]
        # stwórz nową nazwę fragmentu ściezki: ?<name> - zgodny ze wzorcem regexp. 
        # Dodajemy także flagi \w+||\d+, które oznaczają, że oczekujemy, że dana wartość będzie albo słowem (\w) albo liczbą (\d)
        # i będzie zawierała co najmniej jeden znak (+)
        el = '(?<' + name+ '>\w+||\d+)'
      end

      # zapisz przetrawiony (lub nie) element
      new_route << el
    end

   # połącz na nowo nową ścieżkę znakami /
   new_route.join('/')
end

# Przykład użycia:
route_to_regexp('/user/get/:id')
# => '/user/get/?<id>\w+||\d+'
```

Okej, to teraz trzeba zmusić router, żeby w ogóle obsługiwał zmienne w ścieżkach. Nie będzie to *takie* proste, ponieważ jakby nie było, to nie możemy po prostu zrobić `@@routes['GET'][route]` ze względu na zmienne! Aby dokonać tego porównania stworzymy metodę `compare_route`, która przeleci wszystkie klucze haszu `@@routes` i zwróci pierwszy klucz, do którego przypasuje się żądana ścieżka:

```ruby
def compare_route( path, method )
    found_route = ''
    params = {}

    # Dla każdego klucza (wzorca ścieżki) wykonaj blok
    # (method == POST or GET)
    @@routes[method].keys.each do |route|
      # Zamień analizowaną ścieżkę na formę dla regexpa
      regexp_route = route_to_regexp(route)
      # Stwórz nowy Regexp (ograniczmy wzorzec tak, że musi pasować w 100% od początku (^) do końca ($))
      regexp = Regexp.new( '^' + regexp_route + '$' )

      # Jeśli regexp znalazł jakiś wynik po wykonaniu metody .match dla zadanej ścieżki
      if( match = regexp.match(path) )
        # Zwróć ten wynik
        found_route = route
        # A parametry, które udało się dopasować zamień na formę
        # { zmienna1: wartość1, zmienna2: wartość2 }
        params = Hash[match.names.zip(match.captures)]
        break
      end
    end

    # Zwróć wynik jako tablicę, aby otrzymać od razu dwie zmienne
    # (wynik będzie odbierany jako: route, path = compare_route(ścieżka))
    [ found_route, params ]
end
```

Wszystkie powyższe metody umieszczamy w klasie `Route` i musimy jeszcze zastosować `compare_route` w `initialize`, i zapisać znalezione parametry w instancji:

```ruby
def initialize( request )
    route, params = compare_route( request.path, request.request_method )
    target = @@routes[request.request_method][route].split('#')

    @class  = target[0]
    @method = target[1]
    @params = params
end
```

W tej chwili skończyliśmy pisać router :-)

## 3. Dynamiczne ładowanie klas

Po przeanalizowaniu ścieżki przez router otrzymujemy trzy zmienne: nazwę klasy (String), nazwę metody (String) oraz tablicę parametrów.

### Ładowanie klas przy każdym requeście

W klasyczny sposób użylibyśmy `require 'nazwa_klasy.rb'`, ale kolejne requesty nie spowodowałyby przeładowania klasy, ponieważ użycie `require` zapewnia załadowanie pliku tylko jednokrotnie - raz załadowany nie zostanie załadowany ponownie. Wszelkie zmiany w kodzie klasy wymagałyby restartu serwera, a przy procesie pisania kodu jest to wielce niewygodne.

Użyję `load 'nazwa_klasy.rb'` - każde wywołanie teraz załaduje na nowo klasę, nadpisze starą i dzięki temu serwer zawsze będzie pracował na najnowszym kodzie.

### Wywołanie metody w postaci ciągu znaków, na klasie w postaci ciągu znaków i zapisanie w klasie parametrów

Za całość stworzenie instancji kontrolera będzie odpowiadać metoda `create_instance`:

```ruby
# route - obiekt klasy Route
# params - Hash parametrów przekazanych w requeście (np. /path?param=val)
def create_instance( route, params )
    # Wczytaj plik klasy - nazwa znormalizowana
    # @@wd - working directory, katalog w którym uruchomiony został serwer
    load File.join( @@wd, 'app', route.get_class + '.rb')
    # Za pomocą metody Ruby: Object.const_get, jesteśmy w stanie zamienić ciąg znaków na stałą.
    # Każda klasa jest de facto pewnym rodzajem stałej.
    # Natychmiast na takiej klasie wywołujemy metodę .new, aby stworzyć jej instancję
    object = Object.const_get( route.get_class ).new
    # W zmiennej @params (.params) zapisujemy połączone parametry z requesta i zmienne z route'a
    object.params = params.merge( route.get_params )

    # zwracamy instancję klasy
    object
end
```

Załatwione. Teraz pora wywołać na otrzymanej klasie żądaną metodę przy pomocy metody .send. Argumentem jest nazwa metody w postaci ciągu znaków, którą chcemy wywołać na danym obiekcie:

```ruby
instance = create_instance( route, request.query )
instance.send( route.get_method )
```

## 4. Renderowanie odpowiedzi metody kontrolera w postaci ERB

[ERB](http://www.stuartellis.name/articles/erb/) jest systemem szablonów Rubiego i możemy go użyć do renderowania odpowiedzi JSON.

Aby użyć ERB najpierw trzeba zrozumieć jego działanie.

ERB jest związane z klasą `ERB`, a template (jako String) przekazujemy jako pierwszy parametr tworzonego obiektu (`erb = ERB.new(template)`).

Jednak chcemy używać w template'ach zmiennych instancji (`@zmienna`) i tutaj zaczynają się schody. Okazuje się bowiem, że musimy przekazać do ERB tzw. *binding*. Binding w Rubim to specyficzny mechanizm, pozwalający wykonywać jeden kod, w kontekście drugiego. Innymi słowy - chodzi o wykonanie kodu renderującego ERB w kontekście klasy, do której template przynależy. A wszystko po to, żeby ERB miał dostęp do wszystkich parametrów klasy (zależy nam na zmiennych). W taki właśnie sposób działa Rails! W widokach używamy zmiennych instancji.

Aby to osiągnąć, klasa w kontekście której chcemy wywoływać renderowanie template'u musi udostępnić swój binding. Jeśli klasa nie ma zaprogramowanej funkcji udostępnienia bindingu nie można się do niego dostać.

Weźmy przykładową klasę, którą nazwiemy API. Niech API (i inne klasy) dziedziczy po klasie Controller. Dzięki temu wszystkie nowe kontrolery będą posiadały wspólną metodę `.get_binding`. Przy okazji dorzucimy obsługę parametrów (`@params`), natomiast w zmiennej `@render` kontrolery będą zapisywały, który widok chcą wyrenderować.

```ruby
class Controller
  attr_accessor :params,
                :render

  def get_binding
    binding
  end
end

class API < Controller
end
```

I kompletna funkcja do renderowania widoku:

```ruby
# route - obiekt Route
# instance - instancja klasy żądanej
def render( route, instance )
    # Pobierz binding
    instance_binding = instance.get_binding

    # Jeśli instancja ustawiła zmienną render weź to jako nazwę template'u
    if( instance.render )
      to_render = instance.render
      instance.render = nil
    # Jeśli nie, to użyj nazwy metody jako nazwy template'u
    else
      to_render = route.get_method
    end

    # Zbuduj znormalizowaną ścieżkę do template'u
    view = File.join(@@wd, 'views', route.get_class, to_render + '.erb')
    # Wczytaj template
    view = File.read( view )
    # Przygotuj ERB
    template = ERB.new( view )
    # Wyrenderuj template z bindingiem i zwróć wynik
    result = template.result( instance_binding )
end
```

Przykład użycia:

```erb
# @@wd/views/API/index.erb
<h1><%= @variable %></h1>
```

```ruby
class API < Controller
    def initialize
        @render = 'index'
        @variable = 'Hello world!'
    end
end

route = Route.new(...)
klass = API.new
render(route, klass)
# => <h1>Hello world!</h1>
```

## 5. Odpowiedź w formacie JSON

Wykonanie odpowiedzi w formacie JSON to właściwie jedna z prostszych rzeczy. Wystarczy właściwie... ustawić nagłówek odpowiedzi na odpowiedni:

```ruby
# response to obiekt serwera WEBrick
def respond( route, instance, response )
    response.body = render( route, instance )
    response.status = 200
    response['Content-Type'] = 'application/json'
end
```

## Podsumowanie

Udowodniłem, że stworzenie "podstawy" działania Ruby on Rails nie jest trudne. Jest jedynie podchwytliwe ze względu na niecodziennie mechanizmy.

Teraz **IEIE** wykorzystuje Rails, ze względu na większą ilość możliwości i wyższą jakość wykonania.



