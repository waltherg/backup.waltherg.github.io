# Python `any` and `all` Gotchas

Suppose you want to check that a certain value lies within certain
bounds.
Encoding a set of bounds is easy using a list of lists of
upper bound and corresponding lower bound:

```python
bounds = [[lower, upper], [lower, upper], ...]
```

Take as an example
```python
bounds = [[0,1], [0,2], [1,4]]
```

Testing if a value `val` lies within all `bounds` is straightforward
with the Python built-in `all`:

```python
val = 1
print all(val >= bound[0] and val <= bound[1] for bound in bounds)
```
