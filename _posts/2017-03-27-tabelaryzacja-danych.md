
layout: post
title: YAML  tabelaryzacja danych
tags: [DSP2017, IEIE, Ruby, YAML]
comments: true
excerpt: Krótki post o uporządkowaniu danych związanych z bibliotekami


## Wstęp

Witam. Ten wpis pojawia się z opóźnieniem, wywołanym awarią pieca centralnego ogrzewania w rodzinnym domu mojej partnerki.

W Infinity Engine każdy typ pliku posiada swoją własną strukturę. Oprócz tego używać w projekcie będziemy innych danych, które dają się wpasować w definicję tabeli.

Dotychczas przechowywałem informacje związane z plikami (innymi słowy: dane) w plikach skryptowych. Jest to jakieś rozwiązanie, ponieważ dane są uporządkowane, ale brudzą klasy i kod ogólnie.

## YAML

[YAML](http://yaml.org/) to język przeznaczony do strukturalnego zapisu danych. Oznacza to, że w prostym i czytelnym dla człowieka formacie można zapisać liczby, ciągi znaków, tablice, a wszystko przypisać naturalnym nazwom.

Wybrałem YAML, ponieważ go znam. Jest stosowany w plikach tłumaczeń Rails (i w tym miejscu właśnie miałem z nim pierwszy kontakt) i nie raz widziałem, jak ktoś konfigurację zapisywał w tym formacie. Jest popularny i... fajny!

Weźmy na warsztat nagłówek pliku przedmiotu:

```ruby
@@pattern = {
    file_type: ['04', 'word'],
    file_revision: ['04', 'word'],
    name_ref_unidentified: ['04'],
    name_ref_identified: ['04'],
    used_up_item_file: ['08'],
    item_attributes: ['04', 'boolean array'],
    item_type: ['02'],
    useability: ['04', 'boolean array'],
    item_animation: ['02', 'word'],
    min_level: ['02'],
    min_strength: ['02'],
    min_strength_bonus: ['01'],
    kit_usability_1: ['01'],
    min_intelligence: ['01'],
    kit_usability_2: ['01'],
    min_dexterity: ['01'],
    kit_usability_3: ['01'],
    min_wisdom: ['01'],
    kit_usability_4: ['01'],
    min_consitution: ['01'],
    weapon_proficiency: ['01'],
    min_charisma: ['02'],
    price: ['04', 'number'],
    stack: ['02', 'number'],
    inventory_icon: ['08', 'word'],
    lore_to_identify: ['02'],
    ground_icon: ['08', 'word'],
    weight: ['04', 'number'],
    description_ref_unidentified: ['04'],
    description_ref_identified: ['04'],
    description_icon: ['08', 'word'],
    enchantment: ['04', 'number'],
    extended_header_offset: ['04', 'number'],
    extended_header_count: ['02', 'number'],
    feature_table_offset: ['04', 'number'],
    feature_table_equpping_index: ['02'],
    feature_block_count: ['02', 'number']
}
```

I takiego potworka mogliśmy znaleźć na początku klasy `Item::Header`! Zanim zobaczyliśmy kod musieliśmy przewinąć 40 linii danych.

Docelowa struktura wygląda tak:

```yaml
item:
  header:
    - name: signature
      offset: '0000'
      size: 4
      type: word
    - name: revision
      offset: '0004'
      size: 4
      type: word
    - name: name_unidentified
      offset: '0008'
      size: 4
      type: number
    - name: name_identified
      offset: '000c'
      size: 4
      type: number
    - name: replacement_item
      offset: '0010'
      size: 8
      type: 
    - name: flags
      offset: '0018'
      size: 4
      type: array
    - name: type
      offset: '001c'
      size: 2
      type: 
    - name: usability
      offset: '001e'
      size: 4
      type: array
    - name: animation
      offset: '0022'
      size: 2
      type: 
    - name: min_level
      offset: '0024'
      size: 2
      type: number
    - name: min_strength
      offset: '0026'
      size: 2
      type: number
    - name: min_strength_bonus
      offset: '0028'
      size: 1
      type: number
    - name: usability_kit_1
      offset: '0029'
      size: 1
      type: array
    - name: min_intelligence
      offset: '002a'
      size: 1
      type: number
    - name: usability_kit_2
      offset: '002b'
      size: 1
      type: array
    - name: min_dexterity
      offset: '002c'
      size: 1
      type: number
    - name: usability_kit_3
      offset: '002d'
      size: 1
      type: array
    - name: min_wisdom
      offset: '002e'
      size: 1
      type: number
    - name: usability_kit_4
      offset: '002f'
      size: 1
      type: array
    - name: min_consitution
      offset: '0030'
      size: 1
      type: number
    - name: weapon_proficiency
      offset: '0031'
      size: 1
      type:
    - name: min_charisma
      offset: '0032'
      size: 2
      type: number
    - name: price
      offset: '0034'
      size: 4
      type: number
    - name: stack
      offset: '0038'
      size: 2
      type: number
    - name: icon_inventory
      offset: '003a'
      size: 8
      type: 
    - name: lore
      offset: '0042'
      size: 2
      type: number
    - name: icon_ground
      offset: '0044'
      size: 8
      type: 
    - name: weight
      offset: '004c'
      size: 4
      type: number
    - name: description_unidentified
      offset: '0050'
      size: 4
      type: number
    - name: description_identified
      offset: '0054'
      size: 4
      type: number
    - name: icon_description
      offset: '0058'
      size: 8
      type: 
    - name: enchantment
      offset: '0060'
      size: 4
      type: number
    - name: extended_header_offset
      offset: '0064'
      size: 4
      type: number
    - name: extended_header_count
      offset: '0068'
      size: 2
      type: number
    - name: feature_offset
      offset: '006a'
      size: 4
      type: number
    - name: feature_equipping_index
      offset: '006e'
      size: 2
      type: number
    - name: feature_equipping_count
      offset: '0070'
      size: 2
      type: number
```

Dłuższa, ale o większych możliwościach i przede wszystkim - znaczeni bardziej czytelna. Co najważniejsze: umożliwia nam dodawanie nowych, imiennych pól w wygodny sposób.

Jak taki efekt osiągnąć?

## Ruby Hash -> YAML

Przyznaję się... konwersji tej dokonałem ręcznie. Wynika to z tego, że i tak chciałem wprowadzić pole `offset` (uznałem, że może się przydać, a w oryginalnej strukturze go nie było), i zmienić parę nazw. Dlatego każde zestawienie danych przepisałem ręcznie.

## YAML -> Ruby Hash

Ciekawej sprawa wygląda z załadowaniem YAML-ów do biblioteki **IEIE Core**. Skoro wszystkie tabele znajdują się w jednem folderze (`tables/`), to można załadować wszystkie pliki YAML i zapisać je w stałej (constans) o nazwie `TABLES` dostępnej globalnie. Następnie taką zmienną zamrozić, ale niemożliwa była jej edycja: w Rubim można edytować stałe, jednak wyświetla się wtedy monit. Aby uniemożliwić edycję, stosuje się metodę `Object.freeze`.

Tak więc...

```ruby
require 'yaml'

TABLES = {}

# Dla każdego pliku *.yml w folderze tables, wykonaj blok
Dir.glob(File.dirname(__FILE__) + '/tables/*.yml').each do |table|
  # Niech YAML przetrawi ten plik
  table = YAML.load_file(table)
  # I niech wynik zostanie połączony z globalną tabelą
  # Każdy plik YAML musi posiadać unikalną nazwę roota drzewa
  TABLES.merge!(table)
end

# Zamroź
TABLES.freeze

puts 'Tables generated and freezed.'
```

## Podsumowanie

Przerzucenie się na katalogowanie danych tabelarycznych w osobnych plikach zwiększyło przejrzystość kodu i zwiększyło możliwości czytelnego zapisu danych.

Ta da :-)
