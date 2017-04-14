# Фреймонстры

Для того, чтобы поучаствовать в игре вам нужно написать алгоритм для фреймонстра.

Алгоритм – это простой Node.js модуль, который экспортирует единственную функцию, принимающую на вход текущие координаты фреймонстра (state), частичку карты (scope) и хранилище произвольных данных (context), не стираемое между ходами.

На выходе алгоритм должен вернуть одно из доступных действий в виде строки.

Пример простого алгоритма:

```js
module.exports = (state, scope, context) => {
    const actions = ['left', 'right', 'move', 'boom', 'shoot', 'dig', 'wait'];
    const action = actions[Math.floor(Math.random() * actions.length)];

    context.lastAction = action;

    return action;
}
```

Все фреймонстры играют на одной карте и ходят по очереди. Игра запускает алгоритм фреймонстра с параметрами, получает действие и пытается его выполнить. Расчёт действия ограничен по времени и не может быть больше 200ms (в противном случае фреймонстр останется на месте).

Монстры на карте представлены выданными вам значками и цветами, красной стрелкой указано направление его движения. В течении всей конференции карта будет доступна на телевизоре.

<img width="750" alt="map2" src="https://cloud.githubusercontent.com/assets/4534405/25026159/c26b1680-20be-11e7-83c3-2d3c19e87358.png">

__Ваша основная цель__ – набрать как можно большее количество очков,   
убивая других фреймонстров или раскапывая сокровища.

Игроки, набравшие наибольшее количество очков, получают ценные призы.

Подробное описание параметров и возможных действий смотри ниже.

## Как протестировать алгоритм

1. Установите [**Node.js 7**](https://nodejs.org/en/)
1. Склонируйте или скачайте этот репозиторий
1. Установите зависимости `npm install`
1. Разместите алгоритм в файле [__users/player.js__](./users/player.js)
1. Запустите игру `npm run start`
1. Откройте в браузере http://localhost:3000/
1. Наблюдайте, как ваш алгоритм сражается с двумя ботами [_users/easy-bot.js_](./users/easy-bot.js) и [_users/hard-bot.js_](./users/hard-bot.js)
1. Улучшайте свой алгоритм до тех пор, пока не будете готовы отправить его нам

## Как прислать решение

Если вы ещё не зарегистрировались, то это можно сделать [по ссылке](https://events.yandex.ru/surveys/4726/).

1. Скопируйте код целиком из файла __users/player.js__
1. Разместите код на [Github](https://github.com/) или [Gist](https://gist.github.com/) и пришлите ссылку на почту [dump2017_fronttalks@yandex-team.ru](mailto:dump2017_fronttalks@yandex-team.ru)
1. Или пришлите код прямо в теле письма на почту dump2017_fronttalks@yandex-team.ru

__Решение можно прислать сколько угодно раз!__

## Правила игры

### State (текущее положение фреймонстра)

Приходит _первым параметром_ на вход в функцию и содержит текущие абсолютные координаты вашего фреймонстра и направление его движения. Например:

```js
{ direction: 'left', x: 5, y: 4 }
```

Направления: **up (смотрит вверх), right (смотрит вправо), down (смотрит вниз), left (смотрит влево)**

### Scope (окружение фреймонстра)

Приходит _вторым параметром_ на вход в функцию и содержит частичку карты, которую фреймонстр видит вокруг себя на две клетки в виде двумерного массива. Например:

```js
[ [ 'surface', 'surface', 'surface', 'obstacle', 'enemy-left' ],
  [ 'enemy-left', 'surface', 'surface', 'surface', 'surface' ],
  [ 'enemy-up', 'surface', 'user', 'surface', 'surface' ],
  [ 'surface', 'surface', 'enemy-down', 'obstacle', 'enemy-right' ],
  [ 'surface', 'obstacle', 'enemy-down', 'surface', 'surface' ] ]
```

В центре массива находится сам фреймонстр (user), вокруг него могут быть следующие объекты:
* surface - свободная ячейка карты, куда можно продолжить движение
* obstacle – препятствие
* enemy-left – враг, с указанием направления его движения через «-»

При этом левый верхний угол массива всегда ближе к левому верхнему углу карты,  
а правый нижний – к правому нижнему углу карты.

### Context (персональное хранилище фреймонстра)

Приходит _третьим параметром_ на вход в функцию и позволяет хранить произвольные данные, не стираемые между ходами. Это простой объект, который вы можете дополнять данными по своему усмотрению:

```js
module.exports = (state, scope, context) => {
    const action = 'wait';

    context.lastAction = action;
    context.lastScope = scope;

    return action;
}
```

### Действия

#### wait

Пропустить ход – фреймонстр остаётся на месте и возможно просто поджидает свою жертву.

#### left

Поменять направление своего движения, повернувшись налево.

#### right

Поменять направление своего движения, повернувшись направо.

#### move

Сделать шаг вперёд по направлению своего движения.

* Если впереди свободная клетка – фреймонстр передвигается на эту клекту.
* Если впереди препятствие – фреймонстр остаётся на месте.
* Если впереди враг – фреймонстр съедает его, получая __3 очка__.  Съеденный враг теряет __одно очко__, пропускает __4 хода__ и затем появляется в случайном месте карты.

#### shoot

Выстрелить плазменный шаром через клетку вперёд по направлению своего движения, фреймонстр при этом остаётся на месте.

Если на этой клетке был враг – фреймонстр убивает его, получая __3 очка__.  
Убитый враг теряет __одно очко__, пропускает __4 хода__ и затем появляется в случайном месте карты.

Препятствие или враг прямо перед фреймонстром не мешает выстрелу.

#### boom

Взорваться, уничтожив себя и всех врагов в радиусе __1 клетки__.

За каждого убитого врага, фреймонстр получает __1 очко__, но при этом теряет __8 ходов__ и затем появляется в случайном месте карты.

Убитые враги теряют __одно очко__ , пропускают __4 хода__ и затем появляется в случайном месте карты.

#### dig

Попробовать выкопать клад, фреймонстр при этом остаётся на месте.

Если клад успешно выкопан, фреймонстр получает __150 очков__.
