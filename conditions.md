En Ruby, tout est assimilable à `true` en dehors de `false` et `nil` :

```ruby
true  ? true : false # => true
false ? true : false # => false

""    ? true : false # => true
"foo" ? true : false # => true
0     ? true : false # => true
1     ? true : false # => true
[]    ? true : false # => true

nil   ? true : false # => false
```

Il est préférable d'écrire en ternaire les conditions, mais si cela n'est pas possible, alors le recours aux booléans est envisagable.

La dernière valeur calculée sera toujours renvoyée. Avec `&&`, il s'agira de la dernière valeur si elles sont toutes vraies ou de la première valeur fausse rencontrée. Avec `||` il s'agira de la première valeur vraie rencontrée.

```ruby
1 && 2 && 3 # => 3
1 || 2 || 3 # => 1
1 && nil && 3 => nil
1 && nil && 3 || "foo" => "foo"
```

Supposons que `user.admin` renvoit `true` ou `false` selon que l'utilisateur est le rôle d'administrateur ou non, et que `user.name` renvoit le nom de l'utilisateur (`nil` si celui-ci manque).

```ruby
puts user.admin ? 'Welcome admin' : 'You are not authorized'
puts user.admin && 'Welcome admin' || 'You are not authorized'
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

A noter que `&&` a la priorité sur `||`, tout comme en mathématique les opérations `*` et `/` ont la priorité sur les opérations `+` et `-`. Les écritures ci-dessous sont donc équivalentes :

```ruby
foo || (bar && baz) || (gul && zed)
foo || bar && baz || gul && zed
```
