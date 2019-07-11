En Ruby, tout est assimilable à `true` en dehors de `false` et `nil` :

```ruby
true  ? true : false # => true
false ? true : false # => true

""    ? true : false # => true
"foo" ? true : false # => true
0     ? true : false # => true
1     ? true : false # => true
[]    ? true : false # => true

nil   ? true : false # => false
```
