```
In purely functional programming you don't tell the computer what to do as such
but rather you tell it what stuff is. The factorial of a number is the product
of all the numbers from 1 to that number, the sum of a list of numbers is the
first number plus the sum of all the other numbers, and so on. You express that
in the form of functions. You also can't set a variable to something and then
set it to something else later. If you say that a is 5, you can't say it's
something else later because you just said it was 5. What are you, some kind of
liar? So in purely functional languages, a function has no side-effects. 
```

- referential transparency
- Haskell is lazy
- infinite data structures
- statically typed

See my run of [Learn You A Haskell](learn-you-a-haskell).

# Functional resources
- [Learn You A Haskell](http://learnyouahaskell.com/chapters) (book)
- [Monads and Gonads](https://www.youtube.com/watch?v=b0EF0VTs9Dc) (video) 
- [Haskell is useless](https://www.youtube.com/watch?v=iSmkqocn0oQ) (video)
- [Tail recursion optimisation](https://www.quora.com/What-is-tail-recursion-Why-is-it-so-bad) (article)
- [Wikibooks](https://en.wikibooks.org/wiki/Haskell)
- [QuickCheck](https://wiki.haskell.org/Introduction_to_QuickCheck1)

# Key features
```haskell
-- let <bindings> in <expression>

-- use let
cylinder r h =
  let
    sideArea = 2 * pi * r * h
    topArea = pi * r^2
  in
    sideArea + 2 * topArea

-- using where
cylinder' r h =
  sideArea + 2 * topArea
  where
    sideArea = 2 * pi * r * h
    topArea = pi * r^2

4 * (if 10 > 5 then 10 else 0) + 2  

4 * (let a = 9 in a + 1) + 2  

[let square x = x * x in (square 5, square 3, square 2)]  

let boot x y z = x * y + z in boot 3 4 2 


describeList xs = "The list is " ++ what xs  
    where
			what [] = "empty."  
			what [x] = "a singleton list."  
			what xs = "a longer list." 

-- Maximum

maximum' [] = error "empty list"
maximum' [x] = x
maximum' (x:xs)
  | x > maxTail = x
  | otherwise = maxTail
  where maxTail = maximum' xs

maximum'' [] = error "empty list"
maximum'' [x] = x
maximum'' (x:xs) = max x (maximum'' xs)

-- Replicate

replicate' n x
  | n <= 0 = []
  | otherwise = x:replicate' (n-1) x

-- reverse

reverse' [] = []
reverse' (x:xs) = reverse' xs ++ [x]

-- zip
zip' _ [] = []
zip' [] _ = []
zip' (x:xs) (y:ys) = [(x,y)] ++ zip' xs ys

-- quicksort with let
quicksort [] = []
quicksort (x:xs) =
  let smaller = quicksort [a | a <- xs, a <= x]
      larger = quicksort [a | a <- xs, a > x]
  in smaller ++ [x] ++ larger

-- quicksort with where
quicksort' [] = []
quicksort' (x:xs) =
  smaller ++ [x] ++ larger
  where
      smaller = quicksort' [a | a <- xs, a <= x]
      larger = quicksort' [a | a <- xs, a > x]

-- quicksort inline
quicksort'' [] = []
quicksort'' (x:xs) =
  quicksort'' [a | a <- xs, a <= x]
    ++ [x]
      ++ quicksort'' [a | a <- xs, a > x]
```

> So when trying to think of a recursive way to solve a problem, try to think of
> when a recursive solution doesn't apply and see if you can use that as an edge
> case, think about identities and think about whether you'll break apart the
> parameters of the function (for instance, lists are usually broken into a head
> and a tail via pattern matching) and on which part you'll use the recursive
> call.

> Haskell functions can take functions as parameters and return functions as
> return values. A function that does either of those is called a higher order
> function.

> Putting a space between two things is simply function application.

> Simply speaking, if we call a function with too few parameters, we get back a
> partially applied function, meaning a function that takes as many parameters as
> we left out. Using partial application (calling functions with too few
> parameters, if you will) is a neat way to create functions on the fly so we can
> pass them to another function or to seed them with some data.

```haskell
print $ zipwith' (+) [1..10] [10..20]

-- flip
flip' f = g
  where g x y = f y x
```

# map
> map takes a function and a list and applies that function to every element in
> the list, producing a new list. 

```haskell
-- map
map (^2) [1..10]

-- filter
filter (<1.5) (take 10 [1.1, 1.2..])
filter even [1..20]

-- takeWhile
takeWhile (>0) (filter odd (map (+ 2) [10,9..]))
```

```haskell
-- list of functions
listoffunctions = map (*) [0..]
```

# Lambdas
> Lambdas are basically anonymous functions that are used because we need some
> functions only once. 

```haskell
-- So use lambdas in this way when you want to make it explicit that your
function is mainly meant to be partially applied and passed on to a function as
a parameter.

addThree = \x -> \y -> \z -> x + y + z  
flip' f = \x y -> f y x  
```

# Folds
```haskell
sum' list = (foldl (\acc x -> acc + x) 0 list)

-- Generally, if you have a function like foo a = bar b a, you can rewrite it as
-- foo = bar b, because of currying.

sum'' list = (foldr (\acc x -> acc / x) 1 list)
sum''' = foldl (+) 0
```

> If you reverse a list, you can do a right fold on it just like you would have
> done a left fold and vice versa. Sometimes you don't even have to do that. The
> sum function can be implemented pretty much the same with a left and right fold.
> One big difference is that right folds work on infinite lists, whereas left ones
> don't! To put it plainly, if you take an infinite list at some point and you
> fold it up from the right, you'll eventually reach the beginning of the list.
> However, if you take an infinite list at a point and you try to fold it up from
> the left, you'll never reach an end!

> Folds can be used to implement any function where you traverse a list once,
> element by element, and then return something based on that. Whenever you want
> to traverse a list to return something, chances are you want a fold. That's why
> folds are, along with maps and filters, one of the most useful types of
> functions in functional programming.

> The foldl1 and foldr1 functions work much like foldl and foldr, only you don't
> need to provide them with an explicit starting value. They assume the first (or
> last) element of the list to be the starting value and then start the fold with
> the element next to it. With that in mind, the sum function can be implemented
> like so: sum = foldl1 (+). Because they depend on the lists they fold up having
> at least one element, they cause runtime errors if called with empty lists.
> foldl and foldr, on the other hand, work fine with empty lists. When making a
> fold, think about how it acts on an empty list. If the function doesn't make
> sense when given an empty list, you can probably use a foldl1 or foldr1 to
> implement it.

```haskell
-- implementing standard library functions with folds

-- maximum
maximum' = foldr1 (\x acc -> if x > acc then x else acc)

-- reverse
reverse' = foldl (\acc x -> x:acc) []

-- product
-- product' = foldr1 (*)
product' :: (Num a) => [a] -> a  
product' = foldr1 (*) 

-- filter
filter' f = foldr (\x y -> if f x then x:y else y) []

-- head
head' = foldr1 (\x _ -> x)

-- last
last' = foldl1 (\_ x -> x)
```

## scanr and scanl
```haskell
scanl (+) 0 [1..10]
scanr (+) 0 [1..10]

-- reverse using flip
scanl (flip (:)) [] [3,2,1]
```

## How many elements does it take for the sum of the roots of all natural numbers to exceed 1000?
```haskell
length (takeWhile (<1000) (scanl1 (+) (map sqrt [1..]))) + 1 
```

# Function application
```haskell
> map ($ 3) [(4+), (10*), (^2), sqrt]
[7.0,30.0,9.0,1.7320508075688772]

-- function composition
map (\xs -> negate $ sum $ tail xs) [[1..5],[6..9],[10..20]]
map (negate . sum . tail) [[1..5],[6..9],[10..20]]

print . sum . map (*2) $ [1..10]
```

# Point free style
```haskell
fn x = ceiling (negate (tan (cos (max 50 x))))
fn' = ceiling . negate . tan . cos . max 50

print (sum (takeWhile (<10000) (map (^2) (filter odd [1..]))))
print . sum . takeWhile (<10000) . map (^2) . filter odd $ [1..]
```

> Functors are things that can be mapped over, like lists, Maybes, trees, and
> such. In Haskell, they're described by the typeclass Functor, which has only
> one typeclass method, namely fmap, which has a type of fmap :: (a -> b) -> f a
> -> f b. It says: give me a function that takes an a and returns a b and a box
> with an a (or several of them) inside it and I'll give you a box with a b (or
> several of them) inside it. It kind of applies the function to the element
> inside the box.

```haskell
intersperse '-' "bah
```
