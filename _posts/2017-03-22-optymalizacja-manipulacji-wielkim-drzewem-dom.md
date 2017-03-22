---
layout: post
title: JavaScript - optymalizacja manipulacji wielkim drzewem DOM
tags: [DSP2017, IEIE, JavaScript]
comments: true
excerpt: Droga do zoptymalizowania procesu wyświetlania w DOM wyników filtrowania spośród 10 tysięcy elementów
---

## Wstęp

W interfejsie (HTML/CSS/JS) edytora IEIE chciałem, żeby była wyświetlana lista plików, podzielona wg typu plików. Problemem jest jednak to, że jest to lista plików składająca się z 10 tysięcy pozycji.

Samo wyświetlenie powoduje pewne opóźnienie w działaniu interfejsu (na początku drogi były to prawie 3 sekundy). Jednak używanie takiej listy byłoby **skrajnie niepraktyczne**, więc zaimplementowałem filtrowanie wyników. I tutaj zaczęły się schody. Ponieważ chcę, żeby lista plików reagowała na każdy wpisany w polu znak musi to działać szybko. Trzy sekundy opóźnienia to zdecydowanie za dużo. Na końcu drogi osiągnąłem maksymalne opóźnienie rzędu 300ms, czyli poprawiłem wydajność *dziesięciokrotnie**.

## Sytuacja wyjściowa

### Wygląd komponentu

Wygląd komponentu listy plików przedstawia się następująco:

![Wygląd listy plików](http://i.imgur.com/vuSinsq.jpg)

Mam więc listę plików podzieloną na kategorie odpowiadające typowi pliku (*2DA*, *BAM* itd.).

### Format danych

Listę plików przesyłam z silnika w formacie JSON:

```js
[
  {
    name: '2DA',
    files: [
      'file1',
      '...',
      'file111'
    ]
  },
  {
    name: 'BAM',
    files: [
      'file1',
      '...',
      'file5363'
    ]
  },
  // itd
]
```

### HTML

Listę wyświetlam poprzez wbudowane pętle frameworku *Vue.js*, którego używam w projekcie:

```html
<ul class="filetypes">
  <li class="filetypes__item"
      v-for="filetype in searchResults"
      v-on:click="toggleFiles">
    {{ filetype.name }}
    <span class="small">({{ filetype.files.length }})</span>

    <ul class="filetypes__files">
      <li class="filetypes__files__item"
          v-for="file in filetype.files">
        {{ file }}<span class="smaller">.{{ filetype.name }}</span>
      </li>
    </ul>

  </li>
</ul>
```

### JS

A kod filtrujący jest wywoływany przy każdej zmianie pola wyszukiwania:

```js
filter (filter) {
  let result = []
  let copy
  this.filetypesList.forEach((filetype) => {
    copy = { ...filetype } // Create copy of filetype object
    copy.files = copy.files.filter((file) => {
      if (file.indexOf(filter) === 0) {
        return true
      }
    })
    result.push(copy)
  })
  return result
}
```

Sama zmiana zmiennej polega na wywaleniu całego drzewa i wyrenderowania wszystkiego od nowa. Metoda kopiowania obiektu (`{ ...object }`) została zaproponowana przez [Comandeera](https://comandeer.pl)

W tej chwili renderowanie całości zajmuje 3 sekundy przy każdej zmianie pola.

## Krok pierwszy: ukrywanie zamiast usuwania

Pierwszym krokiem optymalizacji, który zrobiłem, była zmiana sposobu ukrywania/pokazywania odfiltrowanych elementów. Dotychczas działo się to poprzez usunięcie całego drzewa DOM i wyświetlenie tylko właściwych elementów. Teraz wszystkie elementy są jedynie ukrywane, a następnie chciane - pokazywane. Dzięki temu udało się zaoszczędzić 500ms, ale 2.5s opóźnienia to nadal rzecz niedopuszczalna.

```js
filter (filter) {
  $('.filetypes__files__item').hide()
  $('.file').filter(function () {
    return $(this).html().toLowerCase().indexOf(filter.toLowerCase()) === 0
  }).parent().show()
},
```

## Krok drugi: buforowanie wyników wyszukiwania

Pomyślałem, że uda się zaoszczędzić chwilę czasu, poprzez buforowanie wyników filtrowania. Wszak przeszukanie i odfiltrowanie 10 tysięcy elementów zajmuje w tym wszystkim najwięcej czasu, czyż nie?

Otóż nie - i popełniłem duży błąd nie sprawdzając dokładnie, ile każda operacja zajmuje czasu. Jak się później okazało, to DOM dławił działanie programu, a nie filtrowanie (filtrowanie trwało maksymalnie 500ms).

Ale wtedy tego nie wiedziałem, więc napisałem bufor dla wyników wyszukiwania. Jego celem było filtrowanie tylko w zbiorze ograniczonym poprzednim filtrowaniem.

Na przykład: jeśli posiadam wyniki filtrowania dla wzorca: `Pant`, to po wpisaniu `Pante` nie muszę szukać wśród wszystkich elementów, a jedynie wśród elementów odfiltrowanych już przez Pante.

Kod długi, ale stosunkowo prosty w działaniu: zapisywałem każdy wynik filtrowania w pamięci, pod indeksem równym hasłu filtrującemu. Następnie przy nowym filtrowaniu odcinałem od końca zapytania po jednej literce aż znalazłem zapisany wynik filtrowania, a jeśli takiego nie znalazłem - robiłem nowe filtrowanie.

Wyglądało to następująco:

```js
filter (filter) {
  // Jeżeli filter jest pusty, pokaż wszystkie pliki i zresetuj bufor
  if (filter === '') {
    $('.filetypes__files__item').show()
    this.searchBuffer = {}
  } else {
    let $result, match, i, saved
    // Jeśli istnieje bufor, zapisz go
    if (this.searchBuffer[filter]) {
      $result = this.searchBuffer[filter]
      // Oznacz, że skorzystaliśmy z zapisanego wyniku
      saved = true
    } else {
      // Nie istnieje bufor, więc szukamy najbliższego zapisanego buforu odcinając po jednej literce od konca
      for (i = 1; i < filter.length; ++i) {
        match = filter.slice(0, -i)
        if (this.searchBuffer[match]) {
          $result = this.searchBuffer[match]
          break
        }
      }
    }
    // Jeśli nic nie znaleziono, to musimy robić nowe filtrowanie
    if (!$result) {
      $result = $('.filename')
    }
    
    // Jeśli nie skorzystano z zapisanego bufora, przeprowadź filtrowanie
    if (!saved) {
      $positive = $()
      $result.each(function () {
        let $item = $(this)
        if ($item.html().toLowerCase().indexOf(filter) === 0) {
          $positive = $positive.add($item)
        }
      })
    } else {
      $positive = $result
    }
    // Ukryj wyniki nie należące do zbioru pozytywnych
    $result.not($positive).parent().hide()
    // Pokaż wyniki należące do zbioru pozytywnych
    $positive.parent().show()
    // Zapisz bufor
    this.searchBuffer[filter] = $positive
  }
},
```

Tyle. A wprowadzenie bufora pozwoliło zaoszczędzić... **50ms**. Tyle co nic, prawda? Czyli napisanie tego całego kodu nie było nic warte.

## Krok czwarty: debounce

Krokiem czwartym było ograniczenie częstości wyszukiwania poprzez wprowadzenie mechanizmu [debounce](https://davidwalsh.name/javascript-debounce-function). Dzięki temu odstęp między kolejnymi wywołaniami funkcji *filter()* wynosił 500ms (później 200ms). Odciążyłem w ten sposób przeglądarkę, ale nadal pozostał nieprzyjemny *lag* w postaci 2500ms.

## Krok piąty: wszystko idzie do kosza

Jedną z metod programowania jest napisanie kodu, próba naprawienia go, a następnie wywalenie wszystkiego do kosza i napisanie od nowa. Przy drugim, albo trzecim podejściu powstanie już naprawdę dobry kod :-)

Postanowiłem podejść teraz do problemu od innej strony. Najpierw sprawdziłem, co tak naprawdę zawodzi. Okazało się, że była to manipulacja DOMem.

Szybki [research](http://stackoverflow.com/questions/28876947/fast-filter-in-a-list-of-records-with-javascript) dostarczył mi odpowiedzi, że najlepiej będzie dodać cały HTML do DOMu naraz, w postaci jednego ciągu znaków.

### HTML

Czegoś takiego z Vue.js nie zrobię, więc zacznę od zmiany HTMLa:

```html
<ul class="filetypes"></ul>
```

Znacznie schludniej :-)

### Filtrowanie

Przy okazji szukania pomocy w Google znalazłem bibliotekę [fast-filter](https://www.npmjs.com/package/fast-filter). Autor twierdzi, że jest szybsza od oryginału. Patrząc w kod - nie mam pojęcia czemu, ale wierzę, że mówi prawdę. Załączę ją i wykorzystam w projekcie.

### JS

#### filter()

Funkcję *filter()** sprowadziłem do zastosowania *fast-filter* i wywołania funkcji *buildFilterResults* z parametrem w postaci tablicy odfiltrowanych wyników.

```js
filter (filter) {
  if (filter === '') {
    this.buildFilterResults(this.filetypesList)
  } else {
    let filetypes = this.filetypesList
    let filetypesLength = filetypes.length
    let result = []
    for (let i = 0; i < filetypesLength; ++i) {
      let copy = { ...filetypes[i] }
      copy.files = copy.files.fastFilter(function (value) {
        return value.toLowerCase().indexOf(filter) === 0
      })
      result.push(copy)
    }
    this.buildFilterResults(result)
  }
},
```

#### buildFilterResults()

Postanowiłem zoptymalizować co tylko się da. Wykorzystałem stronę [jsPerf](https://jsperf.com/) aby dowiedzieć się, jakie są najwydajniejsze metody rozwiązywania konkretnych zadań.

##### Iteracja

Jednym z zadań jest iteracja przez tablicę wszystkich elementów. Według [testu](https://jsperf.com/for-vs-foreach/37) najszybszym rozwiązaniem jest zastosowanie pętli `for` i zdefiniowanie rozmiaru tablicy w zmiennej (aby nie trzeba było jej sprawdzać przy każdej iteracji).

#### Dodawanie ciągów znaków

Aby zbudować strukturę drzewa HTMl muszę dodać do siebie bardzo dużo stringów. Która metoda okazała się [najszybsza](https://jsperf.com/string-concatenation/47)? Nie spodziewałem się tego, ale okazuje się, że zdefiniowanie zmiennej, a następnie dodawanie do niej ciągów znaków przy pomocy `+=` jest szybsze od dodawania ciągów znaków w szeregu `d = a + b + c`.

#### Kod

Końcowy kod wygląda następująco:

```js
buildFilterResults (filetypes) {
  let filetypesLength = filetypes.length
  let html = ''
  for (let i = 0; i < filetypesLength; ++i) {
    let filesLength = filetypes[i].files.length
    html += '<li class="filetypes__item"><span class="filetypes__item__name">'
    html += filetypes[i].name
    html += ' <span class="small">('
    html += filesLength
    html += ')</span></span>'
    html += '<ul class="filetypes__files">'
    for (let j = 0; j < filesLength; ++j) {
      html += '<li class="filetypes__files__item">'
      html += filetypes[i].files[j]
      html += '<span class="smaller">.'
      html += filetypes[i].name
      html += '</span></li>'
    }
    html += '</ul></li>'
  }
  $('.filetypes').html(html)
},
```

#### Rezultat

Tego także się nie spodziewałem, ale interfejs odpowiada teraz nie w ciągu 2500ms, ale **~300ms** (w najgorszym przypadku). Optymalizacja opłaciła się!

## Konkluzja

Dzięki całej tej przygodzie nauczyłem się, że trzeba dokładnie analizować co ile czasu zajmuje w naszym kodzie, jeśli planujemy optymalizację. Odkryłem także portal *jsPerf*, z którego teraz zdarza mi się korzystać. Optymalizacja jest *cool* i *trendy*, dlatego warto ją implementować. Daje **ogromną** satysfakcję.

Najważniejszym krokiem optymalizacji było ładowanie całego drzewa naraz w postaci stringu, zamiast poszczególnych *node'ów* jeden po drugim.

Ostatecznie zrezygnowałem z implementowania bufora wyników, ponieważ interfejs działa bardzo dobrze, a kod bufora był za długi i zbyt złożony.


