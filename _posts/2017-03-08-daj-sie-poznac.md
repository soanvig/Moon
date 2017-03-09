---
layout: post
title: Daj Się Poznać 2017
tags: [DSP2017, IEIE]
comments: false
excerpt: Konkurs Daj Się Poznać 2017 - <b>Infinity Engine Incredible Editor</b>
---

## Wstęp

Postanowiłem wziąć udział w konkursie [Daj Się Poznać](http://devstyle.pl/daj-sie-poznac/) (edycja 2017). Zdecydowałem się na ten krok, ponieważ chcę spróbować swoich sił w czymś nowym - i nie mam na myśli nowego języka czy technologii, ale udział w konkursie *per se*.

Projekt, który zgłoszę do konkursu, to będzie mój interdyscyplinarny projekt programistyczny: [Infinity Engine Incredible Editor](http://github.com/ie-ie).

## Infinity Engine Incredible Editor

Infinity Engine Incredible Editor (w skrócie *IEIE*) jest przeglądarką i edytorem plików silnika gry Infinity Engine, na którym oparte zostały takie tytuły jak: *Baldur's Gate*, *Icewind Dale* czy *Planescape: Torment*.

Infinity Engine jest narzędziem bardzo starym - pierwsza wersja datowana jest na rok 1998. W związku z tym rozwiązania stosowane do przechowywania danych w plikach są... mało przyjazne. Bowiem każdy plik zapisany jest w formacie binarnym (szesnastkowym), a dane nie są w żaden sposób oznaczone. Ale o tym w następnym wpisie..

Celem projektu IEIE jest stworzenie narzędzia zdolnego do analizowania tych plików, wyświetlania ich w przyjaznej formie (niech obrazek będzie obrazkiem, dźwięk dźwiękiem, a przedmiot - zestawieniem danych) i umożliwienia ich edycji.

Aktualnie na "rynku" znajduje się narzędzie, które umożliwia edycję wielu z rodzajów tych plików - [NearInfinity](https://github.com/NearInfinityBrowser/NearInfinity). W czym IEIE będzie lepsze?

Przede wszystkim IEIE będzie (jest) podzielone na trzy główne moduły:

- Silnik (Core) napisany w języku Ruby, który dostarczy podstawowej funkcjonalności: otwierania, analizowania, edytowania i zapisywania plików Infinity Engine. Silnik dodatkowo zawierać będzie aplikację Ruby on Rails służącą jako API (oparte na protokole HTTP).
- Interfejs (Interface) napisany w językach: HTML, CSS, Javascript; wykorzystywać będzie framework JS: [Vue.js](https://vuejs.org/). Komunikować się będzie z Silnikiem poprzez protokół HTTP i wyświetlać otrzymane dane w przyjazny sposób. Oferować będzie także formularze do edycji zawartości plików.
- Szalon (Theme) napisany w językach: HTML, CSS, Javascript; oferować będzie zbiór klas i narzędzi umożliwiających stworzenie wyglądu strony przypominającego interfejs gry Baldur's Gate (część 1.). Wykorzystywany będzie domyślnie w Interfejsie.

IEIE będzie oferowało konfiguralne (pod maską) środowisko, mogące współpracować z innymi narzędziami (protokół HTTP); w domyślnej konfiguracji zapewniające ciekawy wygląd uprzyjemniający pracę.

Aktualnie IEIE znajduje się już w pewnej fazie rozwoju.
