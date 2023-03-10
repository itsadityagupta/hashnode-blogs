---
title: "Difference between def and val in Scala"
datePublished: Tue Dec 07 2021 06:02:26 GMT+0000 (Coordinated Universal Time)
cuid: ckwvp40uo0a4ktqs16fl4far6
slug: difference-between-def-and-val-in-scala
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1638540004057/N3zutIZ0D.png
tags: code, programming-blogs, scala, 2articles1week

---

Oftentimes, `val` and `def` can get confusing in Scala. Especially when there are no parameters to a function. Let me explain with an example:

```
object HelloWorld {
    val total: Int = 123
    
    def totalDef: Int = {
        println("Inside def")
        456
    }

    def main(args: Array[String]) {
        println("Main: "+total)
        println(totalDef)
    }
}
```
If you notice, the syntax of calling a variable `total` and a function `totalDef` is just the same.

This can get confusing and one can easily think that ***if they are of the same syntax, why not change `def` to `val` or vice-versa?***

Well, you can't. It's because though the syntax is similar, their compilation and execution is totally different.

> A variable is initialized only once at the start of a program whereas a function is initialized every time it is called.

For example:

```
val test = {
           println("Inside Val")
           20
}
println(test)
```
When the above code is executed, the output is:
```
Inside Val
20
```
This is because when initializing a variable, a `block` of statements has to be executed. And if you know that in scala,** the value of a block is the last line of the block.** Hence, while executing the block, `Inside Val` is printed, and then the value of `test` i.e. `20` is printed.

No matter if you use `test`, n number of times, *`Inside Val` will always be printed only once*.
However, if I make it a function and call it more than once:

```
def func = {
           println("Inside Def")
           20
}
println(func)
println(func)
```
The output will be:
```
Inside Def
20
Inside Def
20
```
This is because a function is initialized every single time it is called.

---
I hope now you understand the difference between `val` and `def` in Scala.

Thanks for reading!

You can find me on other platforms here:  [bio.link/itsadityagupta](https://bio.link/itsadityagupta) 