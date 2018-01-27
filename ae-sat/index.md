# Auto-crafting is NP-complete

> **Disclaimer:** I am by no means an expert in this sort of thing so please do correct me if I get things entirely
> wrong! I should also warn you that, whilst I'll try to keep this as accessible as possible, some prior
> programming/comp-sci knowledge is recommended.

> **Note:** There were some holes in the initial version: I've added an update at the bottom.

In this little document we'll show that one can represent Boolean satisfiability problems as Applied Energistics (or
Refined Storage) crafting recipes, and so prove that autocrafting is NP-complete.

Firstly, let's start with some simplistic definitions:
 - NP-complete: Whilst we could witter on about "Non-deterministic polynomial time" or whatever, for all intents and
   purposes this just means "it takes a long time".
 - Boolean satisfiability: a problem which attempts to determine whether there is a collection of variables that will
   result in a given Boolean expression evaluating to true.

## Encoding a Boolean expression
For simplicity's sake, we'll limit our Boolean expressions to *conjunctive normal form*. Whilst it is possible to
represent a less limited set of expressions, this approach makes things simpler without losing any expressive
power. Conjunctive normal form simply means our formulae is composed of several parts:

 - **Variables** ($$a$$, $$b$$, ...): These need little explanation. We will attempt to determine whether each variable
   will be true or false.
 - **Negations** ($$\neg a$$): One can only take negations of variables.
 - **Disjunctions** ($$a \vee \neg b$$): At least one child expression must evaluate to true. Note that every child must
   be a variable or a negation.
 - **Conjunction** ($$(a \vee b) \wedge (c \vee d)$$): All child expressions must evaluate to true. Note that every
   child must be a disjunction.

Now we've got these 4 simple constructs, we now need to find a way to represent them in the system...

###  Variables
Variables are definitely the simplest to represent. Simply chose some unique item and insert *one* into your AE (or RS)
system.

![An item representing a variable](variable.png "An item representing a variable")

Note that all our variables are just differently coloured leather helmets. We'll use these (and other leather armour
pieces) as it allows for a large number of unique items.

### Negation
If our negation of a variable must be true, then we know the original variable cannot be true. Thus we define a crafting
recipe which converts our "variable item" into some unique "negation item". Only one of these can exist in the system at
one time (remember there is only one instance of each variable item).

![A recipe mapping a variable item to its negation](negation.png "A recipe mapping a variable item to its negation")

### Disjunction
For a disjunction to hold, only one of its constitute atoms must hold. Thus for each element in the disjunction we
create a recipe which converts our "atom item" (either a variable or negation item) into some unique item for this
disjunction. As we'll only need this "disjunction item" once, only one atom needs to be true.

![Disjunction represented as 3 recipes](disjunction.png "Disjunction represented as 3 recipes")

### Conjunction
Conjunction are comparatively easy to represent: as all children must evaluate to true, we just create a recipe from all
constituents to some other unique item.

![Conjunction represented as a recipe requiring 3 items](conjunction.png "Conjunction represented as a recipe requiring 3 items")

## Trying it out
Let's start with a pretty simple Boolean expression:

$$
(a \vee \neg b \vee a \neg c) \wedge (a \vee \neg b \vee c)
$$

This only has three variables and two clauses (or disjunctions) and so should be pretty trivial to solve. We'll use our
above rules to express the expression in a items and recipes. Even this trivial example requires 3 initial items and 10
patterns. None the less, we can still go ahead:

![The crafting plan for the above equation](solve.png "The crafting plan for the above equation")

Fantastic! Looking at the result we can conclude that $$a$$ must be true (as it shows up in the recipe), $$b$$ must be
false and the value of $$c$$ is inconsequential. Thankfully this is a valid solution to our expression! There are other
solutions (any value of $$b$$ and $$c$$ are valid) but we only expected AE to return one.

Let's try something more complex. Maybe something with 4 variables and 3 clauses:

$$
(a \vee \neg b \vee \neg d) \wedge (\neg a \vee b \vee \neg c) \wedge (b \vee \neg c \vee d)
$$

![The failing crafting plan for the above equation](fail.png "The failing crafting plan for the above equation")

Now this is an issue, it appears that AE says no solution exists when there clearly is one. Digging deeper, it's decided
$$b$$ should be true (which is fine) but also that $$a$$ should be true *and* false (which is less fine).

So whilst one *should* be able to solve SAT problems with your storage system, the current systems do not provide a
suitable algorithm for doing so: preferring something efficient but incomplete over something complete and slow.

## What now?
I'm going to be honest, the previous paragraph's conclusions leave me a little bit gutted. I really was looking forward
to factoring prime numbers through AE recipes. I think it's fair to say that the only logical conclusion is for us to
write our crafting system, with blackjack and DPLL.

On a more serious note, it is interesting that there are some circumstances where AE will say something is not craftable
when it is. Whether this is an issue in practice is a different matter, but I don't think I've ever concerned myself
with the practical side of things.

If you fancy experimenting yourself, I've put together [a Lua program][lua_cnf] which will spawn in the appropriate
items/patterns using ComputerCraft's command computer. You'll need to provide input in the form of [DIMACS `.cnf`
files][cnf_files]. I'd heartily recommend [this website](http://toughsat.appspot.com/) for generating files for random
files, but remember to keep the numbers small!

I'd like to finish off by thanking demhydraz, who got me going down this rabbit whole in the first place, and has been
immensely useful as a sounding board.

## Update #1
I've noticed an issue with the above "proof" where a variable (or its negation) can only be used in one disjunction:
meaning an expression may be considered unsatisfiable even if there is some solution. A work around for this issue would
be to create two recipes, one mapping your initial variable item to a stack of "truthy items" and another recipe mapping
it to a stack of "falsey items" (representing the negation). This allows you to reuse terms without being able to use
both the negation and intial variable.

I don't believe this will allow AE to solve any formulae it couldn't already, but I will update this post when I have
confirmation either way.

[lua_cnf]: https://gist.github.com/SquidDev/898a9674e412c851c31552e4ced615a6 "cnf.lua ComputerCraft script"
[cnf_files]: https://www.dwheeler.com/essays/minisat-user-guide.html "The .cnf format explained"

<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
