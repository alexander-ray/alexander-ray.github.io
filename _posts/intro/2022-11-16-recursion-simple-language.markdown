---
layout: post
title:  "Learning recursion through a minimal interpreted programming language"
date:   2022-11-16 17:30:00 -0700
tags: recursion interpreter parser cpp
---
I think one of the more common (and interesting) practical use-cases for recursion is to solve a class of problems involving parsing/interpreting/compiling text.

# Language description

For example, you could imagine wanting to create your own small programming language with two operators `+, -` which act on positive integral numbers, and we expect these operators to act like addition and subtraction (though we could make them do whatever we want). We'll say we don't support parentheses, and these operators are _right-associative_, meaning the order of precedence goes right->left.

A valid program in this language of ours might look like
```
1 + 3 + 5
```
(which we'd expect to return 9) or 
```
1 - 2
```
(which we'd expect to return -1) while examples of invalid programs might include
```
a + 1
```
or 
```
-
```
or 
```
1 - 2 -
```
or 
```
1 - - 2
```

Note that we will always assume a single space separates every part of a program.

So, the problem statement is 
> write a program that, given a program in this language saved in a text file, executes that program and returns the result
How do you think you'd approach something like that?

# Breaking it down

The key idea--as with pretty much every problem that lends itself to a recursive solution--is to consider how a given problem can be broken up into subproblems that fundamentally take the same form. To tease out this way of thinking, we'll set about labeling the different program parts with some jargon; note, however, there's nothing really special about the words we're using to label parts of the program, we're just adding labels to communicate about things more succintly.

The very simplest program in our language is a single number. For example,
```
1
```
is a program which should evaluate to the value `1`.

Bringing _operators_ into the mix, the next simplest program in our language is one with a single _operator_ (`+` or `-`) and two arguments (numbers on either side). For example,
```
1 + 2
```
`1`, in this example, is our left _expression_, and `2` is our right _expression_.

For any (valid) program in our language, we can deconstruct it such that it takes this general form. Taking, for example,
```
1 + 2 + 3 + 4
```
let's start with the leftmost operator as the one we're interested in. Adding some parentheses, we can visualize this deconstruction process as
```
1 + (2 + 3 + 4)
```
with the left expression again being `1`, and the right expression being `2 + 3 + 4`. Importantly, since we're assuming the program is valid, every _expression_, in and of itself, is a valid program.

More generally, every valid program in our language is an _expression_, and an expression can either be
```
number
```
(i.e. like our very simplest program above) or
```
expression +|- expression
```

Note that we're using expressions to define expressions; this is a _recursive definition_. Exactly like defining a word by using the same word in english.

At the end of the day, we really only know how to add and subtract with numbers--we don't know how to add 1 to an expression, we know how to add 1 to another number.

So, to successfully _evaluate_ a given program, we need to get to the point of adding numbers together. Going back to our example above,
```
1 + (2 + 3 + 4)
```
the result of the program is equal to the result of the left expression, added to the result of the right expression. In this case, the result of the left expression is just a number, `1`. Halfway there. What's the result of the right expression? As a human being, we'd just say "9," but this is actually doing a few things at once, even if we can do it instinctively. Computers are dumb, and can effectively do only one thing at a time. Luckily we know that `2 + 3 + 4` is an expression in and of itself, so we can restart the same process.

The right expression--itself a valid program--looks like
```
2 + 3 + 4
```
which we can again deconstruct into
```
2 + (3 + 4)
```

Again, the overall expression is the sum of the left and right expressions. The left expression is `2`, which is a number and therefore we're done. The right expression is `3 + 4`, which is closer to something we know how to work with, but not quite there. So we restart, and try to evaluate
```
3 + 4
```

Here, the value of the left expression is `3` and the value of the right expression is `4`. Success--we know how to do this. So this expression (program) evaluates to the value `7`.

We can then start to go back up to our previous expression
```
2 + (3 + 4)
```
and replace the right expression with `7`, which we just found. This then becomes
```
2 + 7
```
and we know that evaluates to `9`. So we take one more step "up" to our original expression
```
1 + (2 + 3 + 4)
```
and replace the right expression with the value we've just computed. This then becomes
```
1 + 9
```
and because both the left and right expressions are numbers, we know how to do this. `1 + 9` is `10`, and therefore the result of the original program
```
1 + 2 + 3 + 4
```
is `10`.

Taking a step back, while this isn't how we mentally do arithmetic, the reason we _need_ to break things down into smaller and smaller problems is because computers cannot do multiple things at once. Computers do exactly what we tell them to do, and we need to tell them to do things one step at a time. A computer does not know how to evaluate `1 + 2 + 3 + 4`, but it _does_ know how to evaluate `1 + 9`, so we're just taking the steps to break things down to the point that a computer (read: our interpreter program) knows what to do with it.

At this point it's important to convince yourself that this procedure will work for any valid program of any size. Given a program
```
expression + expression
```
it doesn't matter how large each expression is; we can always break each expression down into their constituent parts in exactly the same way.

# Implementation

Ok, so, how would we actually write this "evaluator" program? Like, I want to actually use this programming language because I don't have a calculator--how do I do that?

Let's call our programming language "AddySubtractySolver", or "Ass" for short. Programs should be written in `.ass` files, and for the sake of simplicity we'll start by assuming every program is valid. In c++, we might start with something like

```cpp
#include <iostream>
#include <fstream>

using namespace std;

int evaluate(string expression);

int main(int argc, char *argv[]) {
    // derive program file name from command line arguments
    if (argc == 1) {
        cout << "Please provide a file name." << endl;
        return -1;
    }
    if (argc > 2) {
        cout << "Please provide no more than one file to run, found " << argc - 1 << "." << endl;
        return -1;
    }

    string programFileName = argv[1];

    // read program into string
    string program;
    ifstream fin(programFileName);

    if (fin.fail()) {
        cout << "Could not open file " << programFileName << "." << endl;
        return -1;
    }
    getline(fin, program); // assume entire program is on first line

    // evaluate program
    int result = evaluate(program);

    // print result
    cout << result << endl;

    return 0;
}

/**
 * Evaluate [expression], returning the result.
 */
int evaluate(string expression) {
    // TODO

    return 0;
}
```

Here, we read the contents of the file we want to execute (whose name is given as a command line argument) into a string, and call a single `evaluate` function. This function has a single parameter--a string, `expression`--and returns the result of the expression. The return type is `int` because our language only returns integers.

Given what we know at this point, we can guess `evaluate` will be a recursive function, and so we know it will probably call itself at some point. As we know, recursive functions will have some number of _recursive cases_ followed by at least one _base case_. In each _recursive case_, the function will call itself, typically on some subset of the given problem; in each _base case_, we can stop "recursing" and simply return the result.

Returning to our language definition, we know an expression (the input of our function) can be can either be
```
number
```
or
```
expression +|- expression
```

Mapping this to our recursive function, we can make the "number" case our base case. As we found when working through our example, when the given expression is a single number, we know how to evaluate it already. If the number is `1`, the evaluation result is `1`. If the number is `235`, the evaluation result is `235`. We can add this into our program easily enough:

```cpp
int evaluate(string expression) {
    int result = stoi(expression);

    return result;
}
```

And now we can compile our Ass interpreter program like
```bash
g++ -std=c++17 main.cpp -o ass
```
and run it like
```
./ass firstProgram.ass
```

If our `firstProgram.ass` contains `1`, our program prints `1`. If it contains `235`, our program prints `235`. Nice. We have a working programming language! Of course, it only works for the simplest programs. If `firstProgram.ass` contains `3 + 4`, our program returns `3` (because [`stoi`](https://cplusplus.com/reference/string/stoi/), oddly, parses the first number up to the space rather than telling us something is wrong). So we've dealt with the simple (base) case where our given expression is just a number, and now we need to deal with the _recursive_ case where the expression takes the form
```
expression +|- expression
```

This is where, practically, it becomes important that we _assume_ the program is valid, as it lets us write some string parsing code that takes advantage of the strict syntactic requirements of our language. There are many ways to do so, but let's decide we're in the recursive case or the base case depending on whether or not `expression` contains a `+` or `-`; if the expression does not contain either operator, we know we're in the base case, and otherwise we're in the recursive case. In code, this looks like

```cpp
int evaluate(string expression) {
    int result;
    
    // Base case
    // If expression contains neither a + or a -, we assume the expression is a number
    if (expression.find('+') == string::npos && expression.find('-') == string::npos) {
        result = stoi(expression);
    }
    // Recursive case
    // If expression contains either a + or a -, we assume the expression takes the form
    //   expression +|- expression
    else {
        result = ??
    }

    return result;
}
```

So, we now know which case we're in, but we just need to figure out how to deal with recursive expressions. Let's go back to the example of
```
1 + 2 + 3 + 4
```
Following the process by hand, we deconstructed this to be 
```
1 + (2 + 3 + 4)
```

where we separate the left expression (`1`), the operator (`+`), and the right expression (`2 + 3 + 4`). In practice, this deconstruction is a bit of string manipulation over the `expression` argument which, again, relies on the strict syntax of our language. In code, we can deconstruct our expression in the recursive case like so

```cpp
// Parse expression into left, operator, and right

// Left expression is everything up until the first space
int currentPosition = 0;
int firstSpacePosition = expression.find(' ');
string leftExpression = expression.substr(currentPosition, firstSpacePosition);

// Operator is everything between the first space and the second space
// We assume operators (+ and -) are only a single character
currentPosition = firstSpacePosition + 1;
string op = expression.substr(currentPosition, 1);

// Right expression is the remainder of the expression 
currentPosition = currentPosition + 2;
string rightExpression = expression.substr(currentPosition);
```

So even though it's not pretty, it'll do for our purpose here. We now have our left expression, the operator, and the right expression in separate string variables. If, for a moment, we assume the operator is `+` and therefore we want to do addition, we know that the result of the overall expression should be the result of the left expression _plus_ the result of the right expression. How do we evaluate expressions? We have our function `evaluate` to do that! So, if we assume the operator is `+`, we can say
```cpp
result = evaluate(leftExpression) + evaluate(rightExpression);
```

Pausing here for a moment, let's really revel in the beauty of what's going on here. Our function was passed an `expression` as an argument, and we're making just about the simplest statement possible about an arithmetic expression: the result of the sum is the _thing_ on the left plus the _thing_ on the right. That's it. `evaluate` is doing practically no actual work, and still it's able to compute the result as it passes itself smaller subproblems.

If the operator is `-` rather than `+`, of course we want to perform subtraction. The structure of the problem doesn't change. So adding a final if-statement to our program, we're left with the following:

```cpp
#include <iostream>
#include <fstream>

using namespace std;

int evaluate(string expression);

int main(int argc, char *argv[]) {
    // derive program file name from command line arguments
    if (argc == 1) {
        cout << "Please provide a file name." << endl;
        return -1;
    }
    if (argc > 2) {
        cout << "Please provide no more than one file to run, found " << argc - 1 << "." << endl;
        return -1;
    }

    string programFileName = argv[1];

    // read program into string
    string program;
    ifstream fin(programFileName);

    if (fin.fail()) {
        cout << "Could not open file " << programFileName << "." << endl;
        return -1;
    }
    getline(fin, program); // assume entire program is on first line

    // evaluate program
    int result = evaluate(program);

    // print result
    cout << result << endl;

    return 0;
}

/**
 * Evaluate [expression], returning the result.
 */
int evaluate(string expression) {
    int result;
    
    // Base case
    // If expression contains neither a + or a -, we assume the expression is a number
    if (expression.find('+') == string::npos && expression.find('-') == string::npos) {
        result = stoi(expression);
    }
    // Recursive case
    // If expression contains either a + or a -, we assume the expression takes the form
    //   expression +|- expression
    else {
        // Parse expression into left, operator, and right

        // Left expression is everything up until the first space
        int currentPosition = 0;
        int firstSpacePosition = expression.find(' ');
        string leftExpression = expression.substr(currentPosition, firstSpacePosition);

        // Operator is everything between the first space and the second space
        // We assume operators (+ and -) are only a single character
        currentPosition = firstSpacePosition + 1;
        string op = expression.substr(currentPosition, 1);

        // Right expression is the remainder of the expression 
        currentPosition = currentPosition + 2;
        string rightExpression = expression.substr(currentPosition);

        // Evaluate expression
        
        // If we're doing addition, result is the sum of the left and right subexpressions
        if (op == "+") {
            result = evaluate(leftExpression) + evaluate(rightExpression);
        }
        // If we're not doing addition, we assume the operator is "-" and the result is the difference of the left and right subexpressions
        else {
            result = evaluate(leftExpression) - evaluate(rightExpression);
        }
    }

    return result;
}
```

This is a fully functional interpreter for Ass! Throwing it some basic tests, we can verify things work as we want them to

| Program | Result |
| --- | ----------- |
| 1 | 1 |
| 28345 | 28345 |
| 1 + 1 | 2 |
| 1 + 2 + 3 + 4 | 10 |
| 300 + 700 + 2000 | 3000 |
| 5 - 3 | 2 |
| 2 - 3 + 1 | -2 |

# What?

This line:
```cpp
result = evaluate(leftExpression) + evaluate(rightExpression);
```
is the "magic," but as with everything, if it seems like magic that's because we're not following step by step.

Generically, writing out the _stack_ of recursive function calls is a good way to build intuition for how things are actually working. Going back to our faithful example of a program 
```
1 + 2 + 3 + 4
```
let's start by calling
```
evaluate("1 + 2 + 3 + 4")
```
We fall into the recursive case, because the string "1 + 2 + 3 + 4" contains one of `+` or `-`. From there, we split out our expression into a `leftExpression` (`1`), `op` (`+`), and `rightExpression` (`2 + 3 + 4`). We're doing addition, so we have
```
evaluate("1 + 2 + 3 + 4") -> evaluate("1") + evaluate("2 + 3 + 4")
```
and we can continue with this process for each subsequent function call.
```
evaluate("1")
```
falls into the base case (because the string does not contain `+` or `-`). We just return "1" as an integer using `stoi`, and our call stack becomes
```
evaluate("1 + 2 + 3 + 4") -> evaluate("1") + evaluate("2 + 3 + 4")
    evaluate("1") -> 1
```
Next,
```
evaluate("2 + 3 + 4")
```
falls into the recursive case, and is split into `leftExpression` (`2`), `op` (`+`), and `rightExpression` (`3 + 4`). Our call stack is now
```
evaluate("1 + 2 + 3 + 4") -> evaluate("1") + evaluate("2 + 3 + 4")
    evaluate("1") -> 1
    evaluate("2 + 3 + 4") -> evaluate("2") + evaluate("3 + 4")
```
We can continue this process to get
```
evaluate("1 + 2 + 3 + 4") -> evaluate("1") + evaluate("2 + 3 + 4")
    evaluate("1") -> 1
    evaluate("2 + 3 + 4") -> evaluate("2") + evaluate("3 + 4")
        evaluate("2") -> 2
        evaluate("3 + 4") -> evaluate("3") + evaluate("4")
```
and finally
```
evaluate("1 + 2 + 3 + 4") -> evaluate("1") + evaluate("2 + 3 + 4")
    evaluate("1") -> 1
    evaluate("2 + 3 + 4") -> evaluate("2") + evaluate("3 + 4")
        evaluate("2") -> 2
        evaluate("3 + 4") -> evaluate("3") + evaluate("4")
            evaluate("3") -> 3
            evaluate("4") -> 4
```

We're now at the point we can make no more calls to `evaluate`, and we can start substituting values back into our call stack.

```
evaluate("1 + 2 + 3 + 4") -> evaluate("1") + evaluate("2 + 3 + 4")
    evaluate("1") -> 1
    evaluate("2 + 3 + 4") -> evaluate("2") + evaluate("3 + 4")
        evaluate("2") -> 2
        evaluate("3 + 4") -> evaluate("3") + evaluate("4") -> 3 + 4 -> 7
            evaluate("3") -> 3
            evaluate("4") -> 4
```
leads to
```
evaluate("1 + 2 + 3 + 4") -> evaluate("1") + evaluate("2 + 3 + 4")
    evaluate("1") -> 1
    evaluate("2 + 3 + 4") -> evaluate("2") + evaluate("3 + 4") -> 2 + 7 -> 9
        evaluate("2") -> 2
        evaluate("3 + 4") -> evaluate("3") + evaluate("4") -> 3 + 4 -> 7
            evaluate("3") -> 3
            evaluate("4") -> 4
```
and finally
```
evaluate("1 + 2 + 3 + 4") -> evaluate("1") + evaluate("2 + 3 + 4") -> 1 + 9 -> 10
    evaluate("1") -> 1
    evaluate("2 + 3 + 4") -> evaluate("2") + evaluate("3 + 4") -> 2 + 7 -> 9
        evaluate("2") -> 2
        evaluate("3 + 4") -> evaluate("3") + evaluate("4") -> 3 + 4 -> 7
            evaluate("3") -> 3
            evaluate("4") -> 4
```

There's no one best way to write these out, but most people will develop a convention that makes sense to them after walking through the process a few times.

And now, it's important to note that this is _precisely_ the process we went through by hand originally. The only difference is that we're writing things out in a way that includes more c++-y specifics about how we parse and recursively evaluate expressions.

# Some asides
## Associativity
Let's take a look at the final test case we ran:
```
2 - 3 + 1
```
which returns a value of -2. This isn't what we'd expect, right? If I asked you to tell me what two minus three plus one is, you'd say "go get a calculator," or maybe "0" if you were in a generous mood. But our program returns `-2`--why?

This is because we've specified our language is _right-associative_, but conventionally we perform arithmetic left-associatively for operators of the same precedence (in the sense of [PEMDAS](https://en.wikipedia.org/wiki/Order_of_operations#Mnemonics)). We can think about this as how we mentally place parentheses; for `2 - 3 + 1` we conventionally (mentally) place parentheses as `(2 - 3) + 1`, where we evaluate from left-to-right (hence, left-associative).

Our language is acting as if it's placing parentheses like `2 - (3 + 1)`; we're evaluating from right-to-left. This is a bit counterintuitive because it _feels_ like our program is starting at the left and going to the right, because we start at the leftmost operator. 

The reason our language is still right-associative can be seen in our written-out call stack. 
```
evaluate("1 + 2 + 3 + 4") -> evaluate("1") + evaluate("2 + 3 + 4") -> 1 + 9 -> 10
    evaluate("1") -> 1
    evaluate("2 + 3 + 4") -> evaluate("2") + evaluate("3 + 4") -> 2 + 7 -> 9
        evaluate("2") -> 2
        evaluate("3 + 4") -> evaluate("3") + evaluate("4") -> 3 + 4 -> 7
            evaluate("3") -> 3
            evaluate("4") -> 4
```
The very first expression being _evaluated_ is `3 + 4`--`evaluate("3 + 4")` is lower in the call stack, and thus gets evaluated before subsequent operations. Another way to think about this is that since `evaluate("2 + 3 + 4")` is broken into `evaluate("2") + evaluate("3 + 4")`, `evaluate("3 + 4")` _must_ be evaluated to successfully evaluate `evaluate("2 + 3 + 4")`.

If we wanted to make our language left-associative, one way to do so would be to split expressions at the rightmost operator. For example, rather than splittting `evaluate("2 + 3 + 4")` into `evaluate("2") + evaluate("3 + 4")`, we could split it into `evaluate("2 + 3") + evaluate("4")` and we get a more traditional sense of precedence.

## Parsing
A major reason our final program is so messy (setting aside the fact we also don't really use helper functions, ostensibly for the sake of narrative) is that we're essentially combining the _parsing_ of the program with the _executing_ of the program. 

In our program, our parsing step is effectively the following chunk of code
```cpp
// Parse expression into left, operator, and right

// Left expression is everything up until the first space
int currentPosition = 0;
int firstSpacePosition = expression.find(' ');
string leftExpression = expression.substr(currentPosition, firstSpacePosition);

// Operator is everything between the first space and the second space
// We assume operators (+ and -) are only a single character
currentPosition = firstSpacePosition + 1;
string op = expression.substr(currentPosition, 1);

// Right expression is the remainder of the expression 
currentPosition = currentPosition + 2;
string rightExpression = expression.substr(currentPosition);
```
where we're turning the raw expression string into the left expression, operator, and right expression. We've buried this deep in our `evaluate` function, whose job is ostensibly to _run the whole program_. If we wanted to either extend our language (to support more operators, different syntax, etc) or otherwise improve our interpreter (to handle invalid programs, leniency in syntax, etc) this would very quickly become far too much to handle within our core execution algorithm (`evaluate`).

Typically, in an interpreter, parsing happens as an initial step before we actually try to evaluate (run) the program. Here, we're parsing strings into other strings, but most parsers will make use of classes and related object-oriented programming features to be able to sensibly represent more complex expressions and operations.