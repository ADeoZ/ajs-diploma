[![Build status](https://ci.appveyor.com/api/projects/status/ai95wt09ag061ygm?svg=true)](https://ci.appveyor.com/project/ADeoZ/ajs-diploma)

# Retro Game

## DEMO

https://adeoz.github.io/RetroGame/

## Содержание

* [Используемые технологии](#tech)
* [Описание](#descr)
* [Структура приложения](#struct)
* [Механика](#mech)
  * [Ход игры](#progress)
* [Персонажи](#characters)
  * [Стартовые характеристики](#spec)
  * [Level Up](#levelup)
  * [Генерация на поле](#generate)
* [Локации](#locations)

## <a name="tech">Используемые технологии

* JavaScript (ООП)
* Webpack
* Babel
* Jest

## <a name="descr">Описание</a>

Двухмерная игра в стиле фэнтези, где игроку предстоит выставлять своих персонажей 3 классов против персонажей нечисти. После каждого раунда, восстанавливается жизнь уцелевших персонажей игрока и повышается их уровень. Игра проводится на 4 локациях с разными условиями генерации персонажей. Возможно сохранение и загрузка игры.

## <a name="struct">Структура приложения</a>

1. GamePlay - класс, отвечающий за взаимодействие с HTML-страницей
1. GameController - класс, отвечающий за логику приложения
1. Character - базовый класс персонажей
1. GameState - объект, который хранит текущее состояние игры
1. GameStateService - объект, который взаимодействует с текущим состоянием
1. PositionedCharacter - Character, привязанный к координате на поле.
1. Team - класс для команды (набор персонажей), представляющих компьютер и игрока
1. EnemyTeam - класс для команды компьютера, отвечающий за принятие решений
1. generators - модуль, содержащий вспомогательные функции-генераторы для генерации команды и персонажей
1. utils - прочие вспомогательные функции

## <a name="mech">Механика</a>

Размер поля фиксирован (8x8). Направления движения: по вертикали, по горизонтали, по диагонали. Допустимо перешагивать через персонажей.  
Персонажи разного типа могут ходить на разное расстояние:
* Мечники/Скелеты - 4 клетки в любом направлении
* Лучники/Вампиры - 2 клетки в любом направлении
* Маги/Демоны - 1 клетка в любом направлении

![](https://i.imgur.com/yp8vjhL.jpg)

Дальность атаки так же ограничена:
* Мечники/Скелеты - атака только соседней клетки
* Лучники/Вампиры - ближайшие 2 клетки
* Маги/Демоны - ближайшие 4 клетки

Клетки считаются "по радиусу", допустим для мечника зона поражения будет выглядеть вот так:

![](https://i.imgur.com/gJ8DXPU.jpg)

Для лучника(отмечено красным):

![](https://i.imgur.com/rIINaFD.png)

### <a name="progress">Ход игры</a>

Игрок и компьютер последовательно выполняют по одному игровому действию, после чего управление передаётся противостоящей стороне. Как это выглядит:
1. Выбирается собственный персонаж (для этого на него необходимо кликнуть левой кнопкой мыши)
2. Далее возможен один из двух вариантов:
    1. Перемещение: выбирается свободное поле, на которое можно передвинуть персонажа (для этого на поле необходимо кликнуть левой кнопкой мыши)
    2. Атака: выбирается поле с противником, которое можно атаковать с учётом ограничений по дальности атаки (для этого на персонаже противника необходимо кликнуть левой кнопкой мыши)
    
Уровень завершается выигрышем игрока тогда, когда все персонажи компьютера погибли.  
Игра заканчивается тогда, когда все персонажи игрока погибают.  
Баллы, которые набирает игрок за уровень равны сумме жизней оставшихся в живых персонажей. Они записываются в рекорд игры. Самое большое количество набранных баллов сохраняется.

## <a name="characters">Персонажи</a>

Игрок:
* Мечник (swordsman)
* Лучник (bowman)
* Маг (magician)  

Компьютер:  
* Демон (daemon)
* Скелет (undead)
* Вампир (vampire)

### <a name="spec">Стартовые характеристики (атака/защита)</a>

* Мечник - 40/10
* Лучник - 25/25
* Маг - 10/40
* Демон - 10/40
* Скелет - 40/10
* Вампир - 25/25

### <a name="levelup">Level Up</a>

* На 1 повышает поле level автоматически после каждого раунда
* Показатель health приводится к значению: текущий уровень + 80 (но не более 100). Т.е. если у персонажа 1 после окончания раунда уровень жизни был 10, а персонажа 2 - 80, то после levelup:
    * персонаж 1 - жизнь станет 90
    * персонаж 2 - жизнь станет 100
* Повышение показателей атаки/защиты также привязаны к оставшейся жизни по формуле: `attackAfter = Math.max(attackBefore, attackBefore * (1.8 - life) / 100)`, т.е. если у персонажа после окончания раунда жизни осталось 50%, то его показатели улучшаться на 30%. Если же жизни осталось 1%, то показатели никак не увеличаться.

### <a name="generate">Генерация на поле</a>

Персонажи генерируются рандомно в столбцах 1 и 2 для игрока и в столбцах 7 и 8 для компьютера:

![](https://i.imgur.com/XqcV1uW.jpg)

## <a name="locations">Локации</a>

### Level 1: prairie

У игрока генерируются два персонажа: (случайным образом - типов `Bowman` и `Swordsman`) с уровнем 1, характеристики соответствуют таблице характеристик (см. раздел ниже).
У компьютера генерируется произвольный набор персонажей в количестве 2 единиц.

### Level 2: desert

У игрока повышается level игроков на 1 + восстанавливается здоровье выживших. Дополнительно случайным образом добавляется новый персонаж уровня 1.
У компьютера случайным образом генерируются персонажи в количестве, равным количеству персонажей игрока, с уровнем от 1 до 2.

### Level 3: arctic

У игрока повышается level игроков на 1 + восстанавливается здоровье выживших. Дополнительно случайным образом добавляется два новых персонаж уровня 1 или 2.
У компьютера случайным образом генерируются персонажи в количестве, равным количеству персонажей игрока, с уровнем от 1 до 3.

### Level 4: mountain

У игрока повышается level игроков на 1 + восстанавливается здоровье выживших. Дополнительно случайным образом добавляется два новых персонаж уровня от 1 до 3.
У компьютера случайным образом генерируются персонажи в количестве, равным количеству персонажей игрока, с уровнем от 1 до 4.
  
После прохождения 4 уровня локации начинаются заново с 1 уровня.
