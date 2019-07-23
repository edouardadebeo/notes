## True / False

En Ruby, tout est assimilable à `true` (*truthy*) en dehors de `nil` qui est assimilable à `false` (*falsy*) et, évidemment, de `false`lui-même :

```ruby
def can_you_trust?(value)
  value ? true : false
end

can_you_trust? true  # => true
can_you_trust? ''    # => true
can_you_trust? 'foo' # => true
can_you_trust? 0     # => true
can_you_trust? 1     # => true
can_you_trust? []    # => true

can_you_trust? false # => false
can_you_trust? nil   # => false
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
  return value ? true : false
}

can_you_trust(1)         # => true
can_you_trust([])        # => true
can_you_trust({})        # => true
can_you_trust(0)         # => false
can_you_trust('')        # => false
can_you_trust(null)      # => false
can_you_trust(undefined) # => false
```

## Opérateurs logiques

Lorsqu'on utilise les opérateurs logiques `&&` et `||`, une et une seule des valeurs testées sera renvoyée :
* avec `&&`, la valeur renvoyée sera la première valeur *falsy* ou la dernière valeur si elles sont toutes *truthy* ;
* avec `||`, la valeur renvoyée sera la première valeur *truthy* ou la dernière valeur si elles sont toutes *falsy*.

En Ruby :

```ruby
0 && 1 && 2 && 3 # => 3 (toutes les valeurs sont truthy)
0 || 1 || false # => 1 (0 est la première valeur truthy)
1 && nil && 3 && false => # nil (nil est la première valeur falsy)
```

En Javascript :

```js
0 && 1 && 2 && 3 // => 0 (0 est falsy)
0 || 1 || false // => 1 (1 est la première valeur truthy)
1 && null && 3 && false // => null (null est la première valeur falsy)
```

L'opérateur `&&` a la priorité sur `||`, tout comme en mathématique les opérations `*` et `/` ont la priorité sur les opérations `+` et `-`. Les écritures ci-dessous sont donc équivalentes :

```ruby
a + (b * c) - (d / e)
a + b * c - d / e

a || (b && c) || (d && e)
a || b && c || d && e
```

Il est néanmoins conseillé de laisser les parenthèses pour la lisibilité du code.

## Usages

Il faut éviter d'utiliser les booléans pour enchaîner les actions sauf si l'on est sûre de leur valeur de retour. Par exemple, si l'on écrit un code qui permet de créer un utilisateur et de lui envoyer un mail de bienvenue, ou de lui retourner une erreur si la création a échoué :

```ruby
User.create(params) && send_welcome_mail || render_error
```

Sur le papier, le code fonctionne. Or, si `send_welcome_mail` peut retourner `false` (par exemple en cas d'échec de l'envoie du mail), alors la première partie `User.create(params) && send_welcome_mail` sera `false` donc la partie ) droite `render_error` sera exécuter, alors que l'utilisateur a bien été créé.

Dans ce cas, il faut privilégier l'écriture suivante :

```ruby
User.create(params) ? send_welcome_mail : render_error
```

## Safe navigation

En Ruby, il est possible d'uiliser la méthode `&.` pour enchaîner les déclarations et retourner directement la valeur si elle est *falsy*. Par exemple :

```ruby
User.first&.name&.upcase
```

Si le premier utilisateur n'a pas de valeur pour `name`, alors `User.first&.name` est égal à `nil` et, grpace à la *safe navigation*, le code s'arrêtera à ce moment et ne tentera pas d'éxecuter la suite. Sans la *safe navigation*, toute la ligne se serait exécutée, ce qui aurait renvoyé une erreur : `undefined method 'upcase' for nil:NilClass`.

S'il n'y a aucun utilisateur, `User.first` renvoit `nil`. À nouveau, le code s'arrêtera à ce moment et renverra `nil` plutôt qu'une erreur `undefined method 'name' for nil:NilClass`.

La *safe navigation* et les opérateurs logiques permettent de rédiger des codes simples :

```ruby
username = user ? user.name&.upcase || 'ANONYME' : nil
```

Avec le code ci-dessus, s'il n'y a pas d'utilisateur, `nil` est renvoyé (avec le ternaire), sinon, s'il n'a pas de `name`, `ANONYME` est retourné (avec l'opérateur `||`), sinon, le nom de l'utilisateur en lettres capitales est retourné.

La *safe navigation* ne doit être utilisée que lorsque la valeur peut être testée. Par exemple, `User` ne peut pas être *truthy* ou *falsy* donc on n'écrira jamais `User&.first` !

Dans le cas des `hash`, pour tester l'existence de la clé, il faut écrire de la manière suivante :

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
* `city` renverra `nil` car la clé `city` n'existe pas dans les propriétés de `address` ;
* `price` renverra `nil` car la clé `product` n'existe pas dans les propriétés de `response`.

## Conseils d'écritures

Quelques astuces pour rendre le code plus lisible.

En Ruby, écrire la *safe navigation* (et plus globalement les méthodes chaînées) sur plusieurs lignes :

```ruby
response
  &.[](:user)
  &.[](:name)
```

En Ruby, écrire les conditions sur plusieurs lignes en regroupant les && ensemble

```ruby
discount = is_student && is_younger_than_25 ||
           is_handicapped ||
           is_a_woman && is_pregnant ||
           is_older_than_70

price = discount ? 25 : 45
```

En Javascript, écrire les ternaires sur plusieurs lignes s'ils sont longs :

```js
// short => inline
a ? b : c

// long => multilines
very_long_condition
  ? very_long_if_truthy_action
  : very_long_if_falsy_action
```

En Javascript, utiliser le fait que `0` soit *falsy* pour tester si un tableau contient des valeurs :

```js
const first_array = [1, 2, 3]
const second_array = []

first_array.length ? 'array contains values' : 'array is empty'  // => array contains values
second_array.length ? 'array contains values' : 'array is empty' // => array is empty
```
