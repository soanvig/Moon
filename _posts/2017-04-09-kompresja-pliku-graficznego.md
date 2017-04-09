---
layout: post
title: Odczytanie i zdekompresowanie plikug graficznego
tags: [DSP2017, IEIE, Ruby]
comments: true
excerpt: Odczytanie binarnego pliku graficznego (bitmapy) skompresowanego metodą RLE (run length encoding).
---

Na wstępnie proszę o wybaczenie. Dawno nie było wpisu, ponieważ nieco się wypaliłem.
I nie mam na myśli projektu IEIE, a jedynie blogowanie. Nigdy nie ciągnęło mnie do blogowania.

## Wstęp

W Infinity Engine mamy do czynienia z dwoma głównymi typami plików graficznych: BMP oraz BAM.

Pliki BMP to dobrze znane (choć w praktyce już nie stosowane) ustandaryzowane bitmapy. Pliki BAM natomiast przypominają w budowie pliki GIF: posiadają klatki (frames) oraz cykle/pętle (cycles). 

Klatka to bitmapa (skompresowana lub nie).

Każda pętla zawiera minimum jedną klatkę. Pętli może być kilka i mogą współdzielić między sobą klatki.

W praktyce pliki BAM mogą być poklatkową animacją lub zespołem powiązanych ze sobą obrazków (a wyświetlanych w postaci pojedynyczych klatek).

I tak plikami BAM są animacje: postaci i potworów, przedmiotów, czarów; elementy interfejsu; a nawet mogą być nimi czcionki.

## Analiza budowy pliku BAM

Oczywiście najprostszym sposobem zapisu bitmapy jest tablica *SZEROKOŚĆxWYSOKOŚĆ*, gdzie każdym elementem tablicy jest piksel zapisany w postaci koloru RGB(A).

Taki zapis jest jednak skrajnie niewydajny. Znacznie lepszym pomysłem jest zastosowanie palety.

### Paleta

Paleta to kolejna tablica, zawierająca maksymalnie 256 kolorów. Po zdefiniowaniu palety wystarczy odwoływać się z obrazka do palety.

Oszczędność bajtów w przypadku takiego rozwiązania zależy od ilości kolorów i od ich powtarzalności na obrazku.

### Przezroczystość

Choć paleta pliku BAM zapisana jest w formacie RGBA, kanał alpha (odpowiadający za przezroczystość) jest niewykorzystywany. Stosuje się zamiast tego tzw. *transparency color*, czyli kolor jednoznaczy z przezroczystością.

W przypadku sztuki filmowej, jak i pliku BAM tym kolorem jest... zielony. Innymi słowy: `rgb(0, 255, 0)`. W rzeczywistości zielony kolor nie będzie przez grę respektowany. Zwyczajnie gra go nie wyświetli, dając w ten sposób efekt przezroczystości.

### Kompresja

Wyobraźmy sobie obrazek, którego połowa zbudowana jest z jednego koloru. Oczywiście można odwoływać się przy każdym pikselu do palety, ale można to też skompresować wykorzystując metodę *RLE - run length encoding*.

Metoda RLE polega na zastąpieniu ciągu bajtów koloru kompresowanego przez dwa bajty: kolor i ilość powtórzeń koloru + 1.

W teorii można zastosować taką kompresję do każdego koloru. W praktyce jest to nieopłacalne i zamiast kompresji otrzymalibyśmy efekt odwrotny. Wynika to z faktu, że metoda RLE jest skuteczna tylko przy ciągach tego samego koloru. W przypadku różnorodności kolorów powoduje to zwiększenie ilości bajtów potrzebnych do zapisu koloru.

Na przykładzie:

Czerwoną linię o długości 5 pikseli i grubości 1 piksela można zapisać bezpośrednio:

```
[1, 1, 1, 1, 1]
```

lub w postaci skompresowanej:

```
[1, 5]
```

gdzie 1 to odniesienie do palety (i koloru czerwonego), a 50 to ilość pikseli w tym kolorze. Zamiast więc stosować 5 bajtów, wykorzystaliśmy 2. To 60% kompresji.

Ta sama sytuacja, ale tym razem kolor czerwony zastąpimy gradientem (każdy piksel w innym kolorze):

```
[1, 2, 3, 4, 5]
```

lub w postaci skompresowanej:

```
[1, 1, 2, 1, 3, 1, 4, 1, 5, 1]
```

Dla koloru 1 - 1 piksel, dla koloru 2 - 1 piksel, dla koloru 3 - 1 piksel itd.

W tym skrajnym przypadku, zamiast skompresować, zwiększyliśmy ilość bajtów z 5 do 10. To -100% kompresji.

### Kompresja w BAM

Aby więc uniknąć zjawiska odwrotnego do kompresji zdecydowano się na kompresowanie jedynie koloru transparentnego, ponieważ okazało się, że średnio grafiki zawierają około 30% koloru transparentnego i zazwyczaj jest on ułożony w sposób ciągły.

## Interpretacja pliku

Pora na odrobinę kodowania. Zacznijmy od rozważenia przypadku klatki nieskompresowanej. Piksele (bajty) zapisane są w postaci szeregu (od lewego górnego narożnika do prawego dolnego narożnika), a obrazek jest kwadratowy. Oznacza to, że analizując dane bajt po bajcie będziemy musieli co `szerokość docelową obrazka` przeskakiwać do nowego wiersza. Kolor przezroczysty tworzymy poprzez zwyczajne nierysowanie piksela.

```ruby
frame.data.each do |val|
  # Pobieramy kolor z palety
  color = @pallete[val]

  # Jeżeli kolor odniesienia (val) nie jest przezroczysty (@pallete[256])
  # lub jeśli nie chcemy tworzyć przezroczystego koloru (!transparent)
  if val != @pallete[256] || !transparent
    # Kolor zapisany jest w formacie blue-green-red, stąd odwrócona kolejność kolorów
    png[column, row] = ChunkyPNG::Color.rgb( color[2].to_i(16), color[1].to_i(16), color[0].to_i(16) )
  end

  # Przeskakujemy do nowej kolumny
  # i jeśli ta kolumna przekracza szerokość obrazka - przeskocz do nowego wiersza, do kolumny 0
  column += 1
  if column == frame[:width]
    column = 0
    row += 1
  end
end
```

Teraz, jeśli obrazek jest nieskompresowany, otrzymamy dokładnie odwozorowaną bitmapę.

### Dekompresja

Dekompresja wymaga napisania dodatkowego kodu (w klasie klatki `BAM::Frame`).

Tutaj rzecz jest nieco bardziej zawiła. Należy się zastanowić, co się dzieje w danych źródłowych. Do stworzenia klatki użyliśmy bajtów w ilości `SZEROKOŚĆxWYSOKOŚĆ`. Ale przecież ramka jest skompresowana. Oznacza to, że zbudowana jest z mniejszej ilości bajtów, niż pobrana do jej stworzenia. Innymi słowy: **ramka zawiera dane innych ramek!**

Czy to źle? **Nie**. Cóż z tego, że ramka zawiera dane innej ramki? Wystarczy, że zdekompresujemy ją normalnie, a potem do końcowego wyniku dekompresji zapiszemy tylko pierwsze `SZEROKOŚĆxWYSOKOŚĆ` bajtów. Reszta może przepaść, ponieważ w przypadku tej konkretnej ramki te dane potrzebne nam nie są, a na pewno są zapisane w innych ramkach.

Jedyną wadą takiego podejścia jest to, że bez dekompresji w ramce mamy zapisane wadliwe dane: część prawidłową, i część nie należącą do danej ramki. W przypadku zapisu takich danych (bez ich wcześniej dekompresji, a następnie kompresji) stworzylibyśmy nieprawidłowe pliki.

```ruby
def decompress!( transparency_index )
    # Jeżeli ramka nie jest skompresowana (bajt w nagłówku ramki), nie dekompresuj
    if !compressed?
      return
    end

    newdata = []

    @data.each_with_index do |val, index|
      # Jeżeli poprzedni bajt był wykorzystany jako przezroczysty oznacza to, że
      # ten bajt jest informacją o ilości pikseli przezroczystych
      if index > 0 && @data[index - 1] == transparency_index
        next
      end

      # Zapisz bajt
      newdata << val

      # Jeżeli bajt jest odniesieniem do przezroczystego koloru
      if val == transparency_index
        # Sprawdź następny bajt, który stanowi o ilości pikseli przezroczystych
        count = @data[index + 1] || 0
        # I dodaj taką ilość przezroczystych pikseli do nowych danych
        newdata.push( *[transparency_index] * count )

        # A następnie oznacz bajt oznaczający ilość pikseli przezroczystych
        # jako WYKORZYSTANY, aby ciąg [0, 0, 1] nie był potraktowany jako:
        # [(0), (0, 0), 1] a jako [(0), (1)]
        @data[index + 1] = -1
      end
    end

    # Zapisz przyciętą część.
    @data = newdata[0, @values[:width] * @values[:height]]

    # I zapisz informację o tym, że ramka jest już zdekompresowana
    @values[:frame_data][0] = '1'
  end
```

## Epilog

Porównanie efektu bez oznaczania bajtu jako wykorzystanego i z oznaczaniem:

![Obrazek bez oznaczania bajtu jako wykorzystany](http://i.imgur.com/Ovw8o14.png)

![Obrazek z oznaczaniem bajtu jako wykorzystany](http://i.imgur.com/r2otWeL.png)

I oba te obrazki zostały wygenerowane powyższym skryptem. Dziękuję za uwagę :)
