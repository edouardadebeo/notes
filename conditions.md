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

Techniques associées :

Vérifier qu'un tableau a des valeurs :
array.length ? 'le tableau contient des valeurs' : 'le tableau est vide'

Vérifier qu'il y a un nombre et autoriser zéro avec l'interpolation :
`${number}` ? 'il y a une valeur (zéro compris)' : 'la valeur est false, blank, null ou undefined'
```

## Opérateurs logiques

Lorsqu'on utilise les opérateurs logiques `&&` et `||`, une et une seule des valeurs testées sera renvoyée :
* avec `&&`, la valeur renvoyée sera la première valeur *falsy* ou la dernière valeur si elles sont toutes *truthy* ;
* avec `||`, la valeur renvoyée sera la première valeur *truthy* ou la dernière valeur si elles sont toutes *falsy*.

En Ruby :

```ruby
0 && 1 && 2 && 3 # => 3 (toutes les valeurs sont truthy)
0 || 1 || false # => 1 (0 est la première valeur truthy)
1 && nil && 3 && false => nil # (nil est la première valeur falsy)
```

En Javascript :

```js
0 && 1 && 2 && 3 # => 3 (0 est falsy)
0 || 1 || false # => 1 (1 est la première valeur truthy)
1 && null && 3 && false => nil # (null est la première valeur falsy)
```

A noter que `&&` a la priorité sur `||`, tout comme en mathématique les opérations `*` et `/` ont la priorité sur les opérations `+` et `-`. Les écritures ci-dessous sont donc équivalentes :

```ruby
foo || (bar && baz) || (gul && zed)
foo || bar && baz || gul && zed
```

***

Il est préférable d'écrire en ternaire les conditions, mais si cela n'est pas possible, alors le recours aux booléans est envisagable.

Supposons que `user.admin` renvoit `true` ou `false` selon que l'utilisateur est le rôle d'administrateur ou non, et que `user.name` renvoit le nom de l'utilisateur (`nil` si celui-ci manque).

```ruby
puts user.admin ? 'Welcome admin' : 'You are not authorized'
puts user.admin && 'Welcome admin' || 'You are not authorized'

puts user.name ? "Hey ${user.name}" : "Hey there..."
puts user.name && "Hey ${user.name}" || "Hey there..."
```

Attention car avec le système `&&/||`, si la deuxième valeur (ici '*Welcome admin*') est fausse, alors la dernière valeur est renvoyée :

```ruby
access_forbidden = user.admin ? false : true # renvoit false pour un administrateur (l'accès n'est pas interdit)
access_forbidden = user.admin && false || true # renvoit true à cause du || true
```

L'usage des booléans est en réalité utile quand les conditions sont plus complexes, par exemple :

```ruby
test = foo > 5 && foo < 10 ||
       foo == 0 && bar == 10 ||
       bar == foo
```
