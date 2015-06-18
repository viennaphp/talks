% I show you my λs - you show me yours!
% Martin Heuschober;
  [CC-BY-NC-SA 4.0](http://creativecommons.org/licenses/by-nc-sa/4.0/)
% June 9<sup>th</sup> 2015

<link rel="stylesheet" href="highlight.js/styles/solarized_light.css">
<link rel="stylesheet" href="reveal.js/css/reveal.css"/>
<script src="highlight.js/highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
<!-- $ pandoc -t revealjs -s % -o index.html -->

<!--

> module Readme where

> import Data.Function (on)
> import Data.List ((\\),subsequences, sortBy, groupBy, intercalate)
> import Data.List.Split (chunksOf)
> import Data.Ord (comparing)

-->

Intro
=====


Me
--

- Martin Heuschober ([epsilonhalbe](https://www.twitter.com/epsilonhalbe))
- Mathematician/Programmer
- [meetup@Lambdaheads](www.meetup.com/Lambdaheads/)
- Work Terna/T-Systems (ab Juli)

Haskell
-------

- pure
- functional
- easier than it looks
- but still intimidating
- worth your while

My λ
====

The Task
--------

Generate Test-Data for a Minesweeper-like Coding Dojo

```markdown
input        |  output
6 5          |
--x--        |  ␣1B1␣
-----        |  12121
x---x        |  B1␣1B
-----        |  11␣11
-----        |  23332
xxxxx        |  BBBBB

```

Base Cases
----------

The minimal cases for such a task would be the board with no and a single entry,
but these cases would be not capturing the full breadth of this problem.

. . .

The heart of this problem would be captured in the field:

```markdown
---       xxx
-?- . . . x?x
---       xxx
```

Where the middle field is the one in question - it can be surrounded by no or at
the maximum 8 `Bombs`.


So how many different 3×3-Fields are there?
-------------------------------------------

Omitting the middle field, we see that 8 fields can either be a bomb or not, so
each fields corresponds to an 8-digit binary number a.k.a. a byte, so we have
$256$-many different minefields. Which leads to the next conclusion:

*"We don't want to write all those test-cases by hand!"*

Haskell to the rescue
=====================

First observation
-----------------

```haskell
λ> :t "abcdefg" -- a String
"abcdefg" :: [Char]
```

. . .

`String`s are just a type synonym for `[Char]` *(list of characters)*.

Second observation
------------------

Lists are a well understood thing with lots of "listy" functions to support it.

```haskell
λ> import Data.List
λ> [1..9]
[1,2,3,4,5,6,7,8,9]
λ> :t subsequences
subsequences :: [a] -> [[a]]
λ> subsequences [1..3]
[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
```

--------------------------------------------------------------------------------

Enumerating all fields we get:

```haskell
123
4 6 => [1,2,3,4,6,7,8,9] = [1..9]\\[5]
789
```

and with a little help of functions we can transform this to our minimal boards

```haskell
λ> import Data.List.Split
λ> :t chunksOf 3
chunksOf 3 :: [a] -> [[a]]
λ> chunksOf 3 [1..9]
[[1,2,3],[4,5,6],[7,8,9]]
λ> putStrLn (unlines $ chunksOf "x-x-x-x-x")
 x-x
 -x-
 x-x
```

--------------------------------------------------------------------------------

```haskell

> minimalBoards = subsequences ([1..9]\\[5]) :: [[Int]]

```

Functions be your friend
------------------------

Though intimidating

```haskell

> fun :: [Int] -> String
> fun lst = map (`aux` lst) [1..9::Int]
>   where aux x lst = if x `elem` lst  then 'x' else '-'

```

They make your life easier

```haskell
λ> map (\x -> x*x) [1..5]
-- map (\x -> x*x) [1,2,3,4,5]
--[1*1,2*2,3*3,4*4,5*5]
  [1  ,4  ,9  ,16 ,25 ]
```

```haskell
λ> 1 `elem` [2,3,1]
True
λ> 4 `elem` [2,3,1]
False
λ> fun [1,5,6,9]
"x---xx--x"
```

Step it Up
==========

Creating Groups
---------------

```haskell

> sortBoards :: [[[Int]]]
> sortBoards = groupBy ((==) `on` length) $
>    sortBy (comparing length) minimalBoards

```

Decomposing
-----------

```haskell
λ> groupBy (==) "aaBBccddddccbaaaaaaaaa"
["aa","BB","cc","dddd","cc","b","aaaaaaaaa"]
λ> let a = it
λ> sortBy (comparing length) a
["b","aa","BB","cc","cc","dddd","aaaaaaaaa"]
```

And Combining
-------------

```haskell

> allPossibleInputs = map (map (unlines . chunksOf 3 . fun)) sortBoards

λ> head allPossibleInputs
["---\n---\n---\n"]
λ> mapM_ putStr $ head allPossibleInputs
 ---
 ---
 ---
```

Last but not least
==================

Preparing
---------

```haskell

> content = [("3 3\n"++x,show j++"-Bomb-input"++show i)
>           | (b,j) <- a , (i,x) <- b]
>         where a = zip (map (zip [0..]) allPossibleInputs) [0..]

λ> take 2 content
[("3 3\n---\n---\n---\n","0-Bomb-input0")
,("3 3\nx--\n---\n---\n","1-Bomb-input0")]
```

Write to File
-------------

```haskell

> main :: IO ()
> main = do putStrLn "Writing a bunch of files to the current directory"
>           mapM_ (\(str,fname) -> writeFile fname str) content
>           putStrLn "Done!"

```

Thank you
---------

Find me at twitter/github @epsilonhalbe

📧 : `map (pred.pred) "octvkp0jgwuejqdgtBiockn0eqo"`

and go to [tryhaskell.org](https://tryhaskell.org) have fun
