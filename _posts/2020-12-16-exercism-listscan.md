---
layout: post
title: Exercism, List.scan and the Twelve Days of Xmas
---


# Introduction
I have been planning to launch a coding blog for quite a while, originally to start in the New Year
, but having found a topic that is relevant to the F# Advent Calendar and it being my birthday 
today, I have decided to launch this early with this post.

Now, one of my coding practices is to play on [exercism](https://exercism.io) when I am between 
projects to improve and keep current my coding  skills. I am currently playing on the 
[F# track](https://exercism.io/tracks/fsharp). In general this blog will provide occasional posts 
on items I found interesting in doing this and other tracks. 

So, I recently completed the [Food Chain](https://exercism.io/tracks/fsharp/exercises/food-chain) 
exercise which, being a [cumulative song ](http://en.wikipedia.org/wiki/Cumulative_song)was, to me,
 was an obvious use of [List.scan](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-listmodule.html#scan), 
but I noticed that no one else's solutions used this. Whilst quite a basic feature of F# collection 
modules, this omission by others seemed worthy of a post.

The Food Chain exercise is one of a number of cumulative song exercises on exercism, another of 
which is Twelve Days (of Xmas) hence the inspiration for this post.
 
*Note that this has brought forward the launch of this blog so not all features will be ready*

**Addition:** Have a strange double scrollbar effect on code blocks that I have spent much time upon. I
had expected by this launch to have sorted out a theme and comments but these have been delayed as a result
of this issue, sorry.

# F# Advent Calendar
This post is part of the [F# Advent Calendar](https://sergeytihon.com/2020/10/22/f-advent-calendar-in-english-2020/). Many thanks to Sergy Tihon for including me, his site blog has been a wonderful source to discover all things F# !

# Challenges and Exercism
This post will discuss three cumulative song exercises on the F# Track on Exercism:

1. House
2. Twelve Days
3. Food Chain

I do suggest that you attempt these on exercism or the equivalent elsewhere before reading 
further but that, of course, is up to you.

There are many on-line training/learning/practice coding sites but I like exercism (although I 
preferred v1 to v2 but all the features of v1 - apart from that great v1 logo - are now back in v2)

It is great way to solve a problem and then examine other solutions. There can be much to learn both from good
solutions and bad ones, learning not just about algorithms, collection module operators etc. but also 
conciseness and readability and so on.

I have quite often discovered a collection module operator that makes a solution more elegant and 
concise. Once learned (mostly) never forgotten. However very few of these discoveries lead to a 
radical update of the algorithm and/or significant performance enhancements or memory/garbage 
collection efficiencies. 

There is one operator that does provide such benefits 
*List.scan* (and its siblings in other collection types) and, I can only recall seeing it be used once to solve a 
challenge.

### List.scan

From the Microsoft documentation:

> Applies a function to each element of the collection, threading an accumulator argument through the computation. Take the second argument, and apply the function to it and the first element of the list. Then feed this result into the function along with the second element and so on. Returns the list of intermediate results and the final result.

It is just fold except it reveals the intermediate state of the accumulator for each iteration. 
This could be useful for construction of, say, technical indicators, or in 
quantitative analysis or time-series statistics for construction of anything as simple as moving averages to highly 
sophisticated ensembles for charting and so on. You will see it in reactive and asynchronous libraries
but the challenges examined here provide a more basic but focused opportunity to see how it works.

The basic pattern to look for is when your are performing transformation of a sequence - usually  a
 map - where the maps calls a function that is or could primarily comprise a fold over a data set. Each generates another fold over that 
*same* data set. These are opportunities to replace this *map - fold* pattern with *scan*. 

The following three challenges do not need much comment once you understand this. These serve to demonstrate
its usefulness and power with some artificial and toy examples. 
 
### House

From the [House](https://exercism.io/tracks/fsharp/exercises/house/solutions) instructions:
> 
> Recite the nursery rhyme 'This is the House that Jack Built'.
> 
>> [The] process of placing a phrase of clause within another phrase of clause is called embedding. It is through the processes of recursion and embedding that we are able to take a finite number of forms (words and phrases) and construct an infinite number of expressions. Furthermore, embedding also allows us to construct an infinitely long structure, in theory anyway.

First I will present a decent solution to this which does not use *scan* then we will see how *scan* can help

(Select the instructions tab within any selected solution for more information, and that 
is the same for the following challenges)

I think this implies the use of scan but most seemed to read it otherwise.

This is currently the most starred solution by [jovanecyk](https://exercism.io/tracks/fsharp/exercises/house/solutions/d5150ea1f5564c0cb43e9b89b46e4971)

```fsharp
let sentences =
    [
        "the house that Jack built"
        "the malt\nthat lay in "
        "the rat\nthat ate "
        "the cat\nthat killed "
        "the dog\nthat worried "
        "the cow with the crumpled horn\nthat tossed "
        "the maiden all forlorn\nthat milked "
        "the man all tattered and torn\nthat kissed "
        "the priest all shaven and shorn\nthat married "
        "the rooster that crowed in the morn\nthat woke "
        "the farmer sowing his corn\nthat kept "
        "the horse and the hound and the horn\nthat belonged to "
    ]

let sentencesForVerse v =
    sentences
    |> List.take v 
    |> List.rev

let rec toVerse = 
    function
    | [] -> ""
    | sentence :: t -> sentence + (toVerse t)

let rhyme = 
    [1..sentences.Length]
    |> List.map sentencesForVerse
    |> List.map toVerse
    |> List.map (sprintf "This is %s.")
    |> String.concat "\n\n"
```
This is a decent soltion. 

This is quite a simple challenge, as this song is highly regular and this is a typical, decent *List.map* 
solution albeit calling a recursive function rather than a *fold* (it can easily be turned into a *fold*)
So how can *List.scan* help?

[My solution:](https://exercism.io/tracks/fsharp/exercises/house/solutions/10d67ba62d0e478f85a571ad602f208c)
```fsharp
let recite startVerse endVerse: string list =    
    let verses =
        [|"house that Jack built.";
          "malt that lay in";
          "rat that ate";
          "cat that killed";
          "dog that worried";
          "cow with the crumpled horn that tossed";
          "maiden all forlorn that milked";
          "man all tattered and torn that kissed";
          "priest all shaven and shorn that married";
          "rooster that crowed in the morn that woke";
          "farmer sowing his corn that kept";
          "horse and the hound and the horn that belonged to"; |]     
    let prefix = "This is the "
    let scanner previous index =
        prefix :: verses.[index] :: " the " ::  List.tail previous

    [1 .. (endVerse - 1)]
    |> List.scan scanner (prefix :: [verses.[0]] )
    |> List.skip (startVerse - 1)
    |> List.map System.String.Concat

```
In this case the initial value of the accumulator is the actual start of the output, in the following 
two challenges this is thrown away, so there are no start and stop offsets in the following. It is very regular 
with only a difference in the first verse which is handled by the initialised accumulator. There is no
state machine/ pattern match required. 

You should be able to see two benefits: the avoidance of the recalculation of prior verses for subsequent 
verses and the re-use of the underlying immutable lists that the prior verses are comprised of. These are 
both more than nice to have features but offers significant algorithmic performance enhancers with little or no loss of readability or conciseness. 
This is an optimisation over *map - fold* but not a premature one.

This is regarded as a medium level challenge, I would say it should be in the easy level.

## Twelve Days

From the [Twelve Day](https://exercism.io/tracks/fsharp/exercises/twelve-days) instructions 

> Output the lyrics to 'The Twelve Days of Christmas'.

Compared to *House* there are some irregularities, which are a reason to encourage *scan* 
rather than not, as we shall see.

In this challenge I will not present other solutions as my own one did not use *scan*! 
So here it is before I updated it!

```fsharp
let recite start stop = 
    let days = [ 
        "a Partridge in a Pear Tree.";
        "two Turtle Doves";
        "three French Hens";
        "four Calling Birds";
        "five Gold Rings";
        "six Geese-a-Laying";
        "seven Swans-a-Swimming";
        "eight Maids-a-Milking";
        "nine Ladies Dancing";
        "ten Lords-a-Leaping";
        "eleven Pipers Piping";
        "twelve Drummers Drumming" ]

    let nth = [| "first"; "second"; "third"; "fourth"; "fifth"; "sixth"; "seventh"; "eighth"; "ninth"; "tenth"; "eleventh"; "twelfth" |]

    let folder = fun acc i curr -> 
        match i with
        | 1 -> curr  
        | 2 -> sprintf "%s, and %s" curr acc 
        | _ -> sprintf "%s, %s" curr acc
    
    let verse n = 
        days 
        |> List.take n
        |> List.fold2 folder (List.head days) [1 .. n]
        |> sprintf "On the %s day of Christmas my true love gave to me: %s"  nth.[n - 1] 
    
    [start .. stop] |> List.map verse
```
This is a clearer example of the *map-fold* pattern I have referred to already. It is quite straightforward to convert this to *scan* and I think there is a useful before and after comparison.

We need, in this case, to convert the built strings into lists that can be cons'd and decapitated 
which have O(1) performance. This also has the benefit we have already seen of using the same immutable lists in following verses. So not only does *scan* stop
 one re-running the same *fold* from scratch for each subsequent verse saving computer cycles but it also saves on heap memory. 

The *folder* function is renamed *scanner* and also collects the rest of the build logic which was in the *verse* 
function - which is now longer needed. The remaining sequential logic was added to the main function pipeline 
which now calls *scanner* rather than a *map*. Since there is no *scan2* function, the initial and accumulator data
 needs to be passed as a tuple rather than two separate arguments

This is the relevant extract from my [current solution:](https://exercism.io/tracks/fsharp/exercises/twelve-days/solutions/ad58eb715c774ffbb2b31ebca90cc08a)
```fsharp
    let scanner = fun (acc,i) curr -> 
        match i, acc with
        | 1 , _         -> curr :: acc  
        | 2 , _ :: tail -> sprintf "%s, and " curr :: tail
        | _ , _ :: tail -> sprintf "%s, " curr :: tail
        |> fun v -> sprintf "On the %s day of Christmas my true love gave to me: " nth.[i - 1] :: v
        |> fun v -> (v, i + 1)
         
    days
    |> List.take stop
    |> List.scan scanner ([], 1)
    |> List.skip start
    |> List.map fst
    |> List.map System.String.Concat
```

I think this solution is not only both more performant and memory efficient but also clearer.  All conditional 
logic is collected in the one pattern match state machine (*scanner*), whilst all the sequential logic is handled 
in the main functions's only function pipeline.

## Food Chain

From the [Food Chain](https://exercism.io/tracks/fsharp/exercises/food-chain/solutions) instructions:

Generate the lyrics of the song 'I Know an Old Lady Who Swallowed a Fly'.
> 
> While you could copy/paste the lyrics, or read them from a file, this problem is much more interesting if you approach it algorithmically.

Now this was interesting because of various complaints about this exercise being similar to others ones. 
Well it is, as you will see with my solution, but the irregularities seems to have caught most out and I searched
 in vain for a solution that was both concise, made minimal uses of the data and reused data in subsequent verses.

Now at this stage of the track, although it is not an advanced challenge, it still is rated as of "medium" 
difficulty and only a few of the over 5000 people who have participated in this track have completed this 
exercise. This also means that most, whatever the experience when they started this track have or should have become 
reasonably proficient in F#. Certainly I have learnt from quite a few in this group on other exercises.

Anyway I ignored long solutions, those that did not construct the swallow lines and looked for any that did 
not map each verse or equivalent. In the 24 or so solutions I did not find any that fit my search! All 
generated each verse from scratch however many were generated. 

Am I missing something here? I don't think so, indeed I think the use of an approach like *scan* was the 
exercise creator's intent.

In the absence of any decent fit, I choose one of this track's two maintainers [Rob Keim](https://exercism.io/tracks/fsharp/exercises/food-chain/solutions/d59af1940b324a8bb006ab2889412c47) to show here

(It is also important to note that these solutions are for practice. It may simply be the case that no one had the time 
to dwell on these to see the type of refactor I am arguing for here. Suffice to say I am presenting what I consider to be 
good solutions, notwithstanding the lack of a *List.scan*. I am not interested in disparaging anyone else's approach here.)

```fsharp
let animals =
    [
        ("fly", "I don't know why she swallowed the fly. Perhaps she'll die.")
        ("spider", "It wriggled and jiggled and tickled inside her.")
        ("bird", "How absurd to swallow a bird!")
        ("cat", "Imagine that, to swallow a cat!")
        ("dog", "What a hog, to swallow a dog!")
        ("goat", "Just opened her throat and swallowed a goat!")
        ("cow", "I don't know how she swallowed a cow!")
        ("horse", "She's dead, of course!")
    ]

let firstLine n =
    let (fst, snd) = animals.[n - 1]
    sprintf "I know an old lady who swallowed a %s.\n%s" fst snd

let middleLine n =
    let first = animals.[n] |> fst
    let second =
        match n with
        | 2 -> "spider that wriggled and jiggled and tickled inside her"
        | _ -> animals.[n - 1] |> fst
    sprintf "She swallowed the %s to catch the %s." first second

let verse n =
    match n with
    | 1 | 8 -> firstLine n
    | _ ->
        let middleLines =
            [ n - 1 .. -1 .. 1 ]
            |> List.map middleLine
            |> List.reduce (sprintf "%s\n%s")
        sprintf "%s\n%s\nI don't know why she swallowed the fly. Perhaps she'll die." (firstLine n) middleLines

let song =
    [1..8]
    |> List.map verse
    |> List.reduce (sprintf "%s\n\n%s")
```

This is a decent and clear solution, although like the other example it does not use fold and it 
gives you some idea of the better other solutions in general. This makes me wonder that one should
seek to refactor recursive functions into fold where possible and then see if one can go further 
than that, which you certainly can in this case. And it is a very significant refactor.

Now, finally, for my solution. You will see how similar it is to Twelve Days, although we now know I did this one with 
*scan* first. Indeed, it is so similar that I really cannot think of much to add. You can see there are now 5 different 
conditions for verse creation but only 3 subsequent paths to complete the verse. There is one extra function call *spider* 
to deal with that scenario. (If it were not for the test *now* requiring a list I would replace the *List.reduce* with a sequence 
expression and yield each verse rather than iteratively build an output list. Also note that some of these examples might be written 
for older versions of the exercise hence the "now" in the previous statement).

So there is an argument that this and Twelve Days are too similar, except everyone, so far (and 
apologies to anyone if I missed their solution) missed, in my view, that this approach should be 
done with *scan* (or equivalent) as this both a better and easier way of looking at this type 
of problem.

Here is [my attempt:](https://exercism.io/tracks/fsharp/exercises/food-chain/solutions/6a63654600424cdd866291f04c7742a9)

```fsharp
let animals = [  "fly","I don't know why she swallowed the fly. Perhaps she'll die."
                 "spider","It wriggled and jiggled and tickled inside her."
                 "bird","How absurd to swallow a bird!"
                 "cat","Imagine that, to swallow a cat!"
                 "dog","What a hog, to swallow a dog!"
                 "goat","Just opened her throat and swallowed a goat!"
                 "cow","I don't know how she swallowed a cow!"
                 "horse","She's dead, of course!" ]
let spider = "spider that wriggled and jiggled and tickled inside her"

let header animal = sprintf "I know an old lady who swallowed a %s." animal
let swallow fst snd = sprintf "She swallowed the %s to catch the %s." fst snd

let buildVerse (prev,verse) (animal, action) = 
    match prev, verse  with
    | ""      ,_          
    | "cow"   ,_          -> header animal :: action :: []
    | "spider",_::_::tail -> header animal :: action :: swallow animal spider :: tail
    | "fly"   ,_::tail       
    | _       ,_::_::tail -> header animal :: action :: swallow animal prev :: tail
    |> fun v -> animal,v
   
let recite start stop: string list = 
    animals
    |> List.take stop
    |> List.scan buildVerse ("",[])
    |> List.skip start
    |> List.map snd
    |> List.reduce (fun a b ->  a @ "" :: b)
```

# Conclusions

A final thought is that maybe many, like me, have moved from C# to F#. Now *Linq* in C# does not have a *scan* operator 
and due to poor type inference discourages much use of *Aggregate* (*fold* here). 

On the other hand I also come from the function-level school of programming from *APL* to *J* where such operators or verbs are vey useful. I alwys missed 
*scan* in *Linq* (which is why I like *MoreLinq*).  

Maybe exercism needs to have a challenge/exercise level comment section where such equivalent insights
can be raised and discussed, rather than only having individual solution commenting. Going to the github is one step too far as it
takes one out of the user facing web site? I will look and see if there are feature requests or PR's for v3 which will
come out shortly.

Anyway it might also be useful to revisit some of my C# solutions and take what I have learnt on this track, to see how it might improve those solutions. Redoing this one
 will also enable  one to see how pattern matching has progressed with C# 8 but that is for another time.

Happy Holidays!

```<language>
>twelvedays recite 1 12
On the first day of Christmas my true love gave to me: a Partridge in a Pear Tree.
On the second day of Christmas my true love gave to me: two Turtle Doves, and a Partridge in a Pear Tree.
On the third day of Christmas my true love gave to me: three French Hens, two Turtle Doves, and a Partridge in a Pear Tree.
On the fourth day of Christmas my true love gave to me: four Calling Birds, three French Hens, two Turtle Doves, and a Partridge in a Pear Tree.
On the fifth day of Christmas my true love gave to me: five Gold Rings, four Calling Birds, three French Hens, two Turtle Doves, and a Partridge in a Pear Tree.
On the sixth day of Christmas my true love gave to me: six Geese-a-Laying, five Gold Rings, four Calling Birds, three French Hens, two Turtle Doves, and a Partridge in a Pear Tree.
On the seventh day of Christmas my true love gave to me: seven Swans-a-Swimming, six Geese-a-Laying, five Gold Rings, four Calling Birds, three French Hens, two Turtle Doves, and a Partridge in a Pear Tree.
On the eighth day of Christmas my true love gave to me: eight Maids-a-Milking, seven Swans-a-Swimming, six Geese-a-Laying, five Gold Rings, four Calling Birds, three French Hens, two Turtle Doves, and a Partridge in a Pear Tree.
On the ninth day of Christmas my true love gave to me: nine Ladies Dancing, eight Maids-a-Milking, seven Swans-a-Swimming, six Geese-a-Laying, five Gold Rings, four Calling Birds, three French Hens, two Turtle Doves, and a Partridge in a Pear Tree.
On the tenth day of Christmas my true love gave to me: ten Lords-a-Leaping, nine Ladies Dancing, eight Maids-a-Milking, seven Swans-a-Swimming, six Geese-a-Laying, five Gold Rings, four Calling Birds, three French Hens, two Turtle Doves, and a Partridge in a Pear Tree.
On the eleventh day of Christmas my true love gave to me: eleven Pipers Piping, ten Lords-a-Leaping, nine Ladies Dancing, eight Maids-a-Milking, seven Swans-a-Swimming, six Geese-a-Laying, five Gold Rings, four Calling Birds, three French Hens, two Turtle Doves, and a Partridge in a Pear Tree.
On the twelfth day of Christmas my true love gave to me: twelve Drummers Drumming, eleven Pipers Piping, ten Lords-a-Leaping, nine Ladies Dancing, eight Maids-a-Milking, seven Swans-a-Swimming, six Geese-a-Laying, five Gold Rings, four Calling Birds, three French Hens, two Turtle Doves, and a Partridge in a Pear Tree.
```
>
