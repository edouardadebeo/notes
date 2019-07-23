## True / False

En Ruby, tout est assimilable à `true` (*truthy*) en dehors de `nil` qui est assimilable à `false` (*falsy*) et, évidemment, de `false` lui-même :

```ruby
def can_you_trust?(value)
  value ? 'truthy' : 'falsy'
end

can_you_trust? true      # => truthy
can_you_trust? ''        # => truthy
can_you_trust? 'foo'     # => truthy
can_you_trust? 0         # => truthy
can_you_trust? 1         # => truthy
can_you_trust? []        # => truthy
can_you_trust? User.last # => truthy (if user exists)

can_you_trust? false     # => falsy
can_you_trust? nil       # => falsy
```

Attention ! *truthy* ne veut pas dire que la valeur est *égale* à `true`, mais qu'elle peut être utilisée comme telle dans une condition :

```ruby
1 == true  # => false
1 == false # => false
1 ? 'truthy' : 'falsy' # => truthy
```

Note : dans d'autres langages, les valeur `0` et `''` sont *falsy*, par exemple en Javascript :

```js
function can_you_trust(value) {
  return value ? 'truthy' : 'falsy'
}

can_you_trust(1)         # => truthy
can_you_trust([])        # => truthy
can_you_trust({})        # => truthy

can_you_trust(0)         # => falsy
can_you_trust('')        # => falsy
can_you_trust(null)      # => falsy
can_you_trust(undefined) # => falsy
```

## Opérateurs logiques

Lorsqu'on utilise les opérateurs logiques `&&` et `||`, une et une seule des valeurs testées sera renvoyée :
* avec `&&`, la valeur renvoyée sera la première valeur *falsy* ou la dernière valeur si elles sont toutes *truthy* ;
* avec `||`, la valeur renvoyée sera la première valeur *truthy* ou la dernière valeur si elles sont toutes *falsy*.

En Ruby :

```ruby
1 && 2 && 3            # => 3     (toutes les valeurs sont truthy)
1 && nil && 3 && false # => nil   (nil est la première valeur falsy)
0 || 1 || false        # => 1     (0 est la première valeur truthy)
false || nil           # => nil   (toutes les valeurs sont falsy)
nil || false           # => false (toutes les valeurs sont falsy)
```

En Javascript :

```js
1 && 2 && 3             // => 3    (toutes les valeurs sont truthy)
1 && null && 3 && false // => null (null est la première valeur falsy)
0 || 1 || false         // => 1    (1 est la première valeur truthy)
null || undefined || 0  // => 0    (toutes les valeurs sont falsy)
0 || undefined || null  // => null (toutes les valeurs sont falsy)
```

L'opérateur `&&` a la priorité sur `||`, tout comme en mathématique les opérateurss `*` et `/` ont la priorité sur les opérateurs `+` et `-`. Les écritures ci-dessous sont donc équivalentes :

```ruby
a + (b * c) - (d / e)
a + b * c - d / e

a || (b && c) || (d && e)
a || b && c || d && e
```

## Booléans VS ternaires

Il faut éviter d'utiliser les booléans pour enchaîner les actions sauf si l'on est sûr de leur valeur de retour. Par exemple, si l'on écrit un code qui permet de créer un utilisateur puis, si la création a fonctionné, de lui envoyer un mail de bienvenue, ou de lui retourner une erreur si la création a échoué :

(note : `User.create` renvoit le `user`, donc *truthy*, ou `false` selon que la création ait fonctionné ou non)

```ruby
User.create(params) && send_welcome_mail || render_error
```

Si `send_welcome_mail` retourne `false` (par exemple en cas d'échec de l'envoie du mail), alors la première partie `User.create(params) && send_welcome_mail` sera `false` donc la partie à droite `render_error` sera exécutée, alors que l'utilisateur a bien été créé.

Il faut donc mieux privilégier l'écriture suivante :

```ruby
User.create(params) ? send_welcome_mail : render_error
```

## Safe navigation

En Ruby, il est possible d'uiliser la méthode `&.` pour enchaîner les déclarations et retourner directement la valeur si elle est *falsy*. Par exemple :

```ruby
User.first&.name&.upcase
```

Si le premier *user* n'a pas de valeur pour `name`, alors `User.first&.name` est égal à `nil` et, grâce à la *safe navigation*, le code s'arrêtera à ce moment et ne tentera pas d'éxecuter la suite. Sans la *safe navigation*, toute la ligne se serait exécutée, ce qui aurait levé une exception : `undefined method 'upcase' for nil:NilClass`.

S'il n'y a aucun *user*, `User.first` renvoit `nil`. À nouveau, le code s'arrêtera à ce moment et renverra `nil` plutôt que de lever une exception `undefined method 'name' for nil:NilClass`.

La *safe navigation* et les opérateurs logiques permettent de rédiger des codes simples :

```ruby
username = user ? user.name&.upcase || 'ANONYME' : nil
```

Avec le code ci-dessus :
* s'il n'y a aucun *user*, `nil` est renvoyé (avec le ternaire) ;
* sinon, si le *user* n'a pas de valeur pour `name`, `ANONYME` est retourné (avec l'opérateur `||`) ;
* sinon, le nom de l'utilisateur en lettres capitales est retourné.

La *safe navigation* ne doit être utilisée que lorsque la valeur peut être testée (être *truthy* ou *falsy*). Par exemple, `User` ne peut pas être *truthy* ou *falsy* donc on n'écrira jamais `User&.first` !

Dans le cas d'un `hash`, pour tester l'existence de la clé, il faut écrire de la manière suivante :

```ruby
response = {
  user: {
    name: 'John'
  },
  address: {}
}

name = response&.[](:user)&.[](:name)&.upcase
city = response&.[](:address)&.[](:city)&.upcase
price = response&.[](:product)&.[](:price)&.upcase
```

* `name` renverra `JOHN` car `response[:user][:name]` existe ;
* `city` renverra `nil` car `response[:address]` existe mais `response[:address][:city]` n'existe pas ;
* `price` renverra `nil` car`response[:product]` n'existe pas.

## Conseils d'écritures

Créer des méthodes intermédiaires pour que les conditions soient nommées et que leur sens soient directement compris :

```js
// sans méthode intermédiaire
if (number % 2 === 0) { // que teste-t-on ?
  // do something
}

// avec méthode intermédiaire
const isEven = number => number % 2 === 0

if (isEven(number)) { // la condition est exiplique
  // do something
}
```

En Ruby, il est possible d'écrire la *safe navigation* (et plus globalement les méthodes chaînées) sur plusieurs lignes :

```ruby
products = { strawberries: 2, pineapple: 0, oranges: 4, bananas: 1 }

products
  &.select { |_, qty| qty.positive? } # => { strawberries: 2, oranges: 4, bananas: 1 }
  &.sort_by { |_, qty| qty }          # => [[:bananas, 1], [:strawberries, 2], [:oranges, 4]]
  &.reverse                           # => [[:oranges, 4], [:strawberries, 2], [:bananas, 1]]
  &.to_h                              # => { oranges: 3, strawberries: 2, bananas: 1 }
  
# => { oranges: 3, strawberries: 2, bananas: 1 }
```

En Ruby, écrire les conditions sur plusieurs lignes en regroupant les && ensemble

```ruby
discount? = student? && younger_than?(25) ||
            handicapped? ||
            woman? && pregnant? ||
            older_than?(70)

price = discount? ? 25 : 45
```

En Javascript, écrire les ternaires sur plusieurs lignes s'ils sont longs :

```js
// short => inline
a ? b : c

// long => multilines
very_long_condition
  ? very_long_if_truthy_action(params)
  : very_long_if_falsy_action(other_params)
```

En Javascript, utiliser le fait que `0` soit *falsy* pour tester si un tableau contient des valeurs :

```js
const first_array = [1, 2, 3]
const second_array = []

first_array.length ? 'array contains values' : 'array is empty'  // => array contains values
second_array.length ? 'array contains values' : 'array is empty' // => array is empty
```
