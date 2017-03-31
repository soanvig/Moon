---
layout: post
title: Jak ugryźć nieistniejące kodowanie znaków?
tags: [DSP2017, IEIE, Ruby]
comments: true
excerpt: Co zrobić, jeśli znaki w pliku zakodowane są w nieistniejącym kodowaniu?
---

## Wstęp

Aby gra mogła istnieć, musi zawierać jakiś tekst. Nie wyobrażam sobie gry bez tekstu. Tak i *Baldur's Gate* posiada tekst :)

W praktyce taki tekst może być *hard-coded* wewnątrz gry, jednak jest to rozwiązanie nierozsądne, ponieważ:

- miesza się logikę z bazą danych (tekstu),
- teksty są rozsiane między wieloma plikami,
- tłumaczenia tekstu są niemożliwe.

W przypadku umieszczenia wszystkich tekstów, jakie w aplikacji są widoczne dla użytkownika, w jednym pliku, tłumaczenie to kwestia wykonania kopii pliku i jego przetłumaczenie. To rzecz oczywista dla każdego programisty, który może się pochwalić *jakimś* doświadczeniem. Tak też rozwiązano sprawę w Infinity Engine - wszystkie teksty znajdują się w pliku **dialog.tlk**.

Jednak ze względu na mnogość języków i występujących w nich znaków diaktrycznych, pojawia się problem zapisania ich (znaków) w postaci bajtowej (wszak bajtem jest każdy znak).

Życiowy problem rozwiązano wymyślając tzw. "*kodowania*". Oprócz domyślnych znaków pojawiających się w [ASCII](https://pl.wikipedia.org/wiki/ASCII) stosuje się dodatkowe bajty (których w ASCII nie zdefiniowano) do zakodowania znaków diaktrycznych. Niestety: sposób kodowania zależy od programisty. Choć teraz standardem jest kodowanie [UTF-8](https://pl.wikipedia.org/wiki/UTF-8), to kiedyś tak nie było. Pewnie dlatego, że dostęp do Internetu nie był powszechny. 

## Więc co jest problemem?

Wspomniałem, że kiedyś nie istniało coś takiego jak *standard kodowania*. A Infinity Engine pochodzi z przełomu tysiącleci. Wtedy też stworzono tłumaczenia gier IE. I każda ekipa tłumacząca stosowała własne kodowanie, które następnie **w jakiś sposób** gra interpretowała (i nie mam pojęcia, gdzie to jest zapisane, jakiego kodowania ma używać gra dla pliku *dialog.tlk*).

I tak może być użyty windows-1250, utf-8, czy inny. Dla każdego z tych kodowań znana jest tablica znaków, więc konwersja bajtów jest prosta (wystarczy je przepuścić przez konwerter używający tablicy). Ruby oferuje potrzebne funkcje do konwersji między kodowaniami.

Niestety, próbując na chybił trafił (najpopularniejsze kodowania), nie potrafiłem poprawnie przekodować znaków z polskiego tłumaczenia *Baldur's Gate*. Postanowiłem sprawdzić, o co chodzi...

> Spodziewany tekst:
> Śmieszni jesteście!
> 
> Wynikowy tekst:
> \235mieszni jeste\244cie!

No tak. Czyli **polskich znaków nie łapie** żadne kodowanie, jakiego jestem w stanie użyć w Rubim. Postanawiam znaleźć kodowanie, w jakim zapisano plik. Szukam więc w sieci, dla jakiego kodowania litera "Ś" przyjmuje wartość "235".

Okazuje się, że... dla żadnego. Po prostu nie istnieje takie kodowanie.

## Rozwiązanie problemu

Jak przekonwertować "nieistniejące" znaki na UTF-8 (standard)? Napisać własny konwerter. Przeglądając różne stringi z gry udało mi się stworzyć tablicę polskich znaków:

```yaml
encodings:
    bg1:
      pl:
        encoding:
        custom:
          229: Ą
          230: Ć
          231: Ę
          232: Ł
          233: Ń
          234: Ó
          235: Ś
          236: Ź
          237: Ż
          238: ą
          239: ć
          240: ę
          241: ł
          242: ń
          243: ó
          244: ś
          245: ź
          246: ż
```

Zastosowałem klucz "bg1", bo nie wiem na razie, jak sprawa wygląda w innych grach opartych na Infinity Engine.

Dobrze, mam tablicę. Teraz funkcja, która przekonwertuje znaki. Oczekiwane zachowanie:

- Wykonaj konwersję dla podanego stringu, według podanej tablicy znaków;
- Jeśli w tablicy zdefiniowano docelowanie kodowanie, użyj go, jeśli nie - użyj UTF-8;
- Jeśli bajt jest zdefiniowany w tablicy znaków dowolnych, zamień go na znak w znanym kodowaniu;
- Przekonwertuj wynik.

No i tak to wygląda:

```ruby
def custom_encode( params = {} )
    table = params[:table] || {}
    encoding = params[:encoding] || table['encoding'] || 'UTF-8'
    custom = params[:custom] || table['custom'] || {}

    # Dla każdego znaku
    self.chars.map do |char|
      char = custom[ char.ord ] if custom[ char.ord ]
      char.force_encoding( encoding )
    end.join
end

# przykład użycia:
"Witaj świecie".custom_encode( CHAR_TABLE )
```

Teraz kwestia użycia tego przy odtwarzaniu stringów i jesteśmy w domu :)

## Podsumowanie

Mam nadzieję, że taki problem jest tylko przy polskim tłumaczeniu, a cała reszta działa poprawnie. Jednak największą tajemnicą jest dla mnie to, **skąd** gra wie, w jakim kodowaniu został zapisany plik?
