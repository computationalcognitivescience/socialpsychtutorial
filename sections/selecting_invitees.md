---
layout: index
title: Dialogue 1 - Individual models
permalink: /dialogue1/
sidebar_link: true
sidebar_sort_order: 4
---

<div id="toc-wrapper" markdown="1">
* Table of contents
{:toc}
</div>

On this page you can find three implementations of variants 4, 5 and 6 of
<span style="font-variant: small-caps;">Selecting invitees</span>. Included
are the formalizations themselves and the code to run a simulation for them.
You need to press the run button to run the simulation. This then also allows
you to change the input of the simulation to explore the behavior of the
models. You are encouraged to explore the simulations to your heart's content.
Afterwards, you can [compare the models through analysis here](/socialpsychtutorial/dialogue1_comparison).


# Selecting Invitees V4

<span style="font-variant: small-caps;">Selecting invitees (version 4)</span>

*Input:* A set $$P$$, subsets $$L \subseteq P$$ and $$D \subseteq P$$ with $$L \cap D = \emptyset$$ and $$L \cup D = P$$, a function $$like: P \times P \rightarrow \{true, false\}$$, and a threshold value $$k$$.

<span>*Output:* $$G \subseteq P$$ such that $$|G\cap D| \leq k$$ and $$|X| + |G|$$ is maximized (where $$X = \{p_i,p_j \in G~|~like(p_i,p_j) = true \wedge i\neq j\}$$).</span>

{% scalafiddle template="Persons", layout="v40" %}
```scala
def si4(P: Set[Person],
        L: Set[Person],
        D: Set[Person],
        like: (Person, Person) => Boolean,
        k: Int): Set[Person] = {
  requirement(L subsetOf P, "L must be a subset of P")
  requirement(D subsetOf P, "D must be a subset of P")
  requirement((L intersect D).isEmpty, "intersection between L and D must be emtpy")
  requirement((L union D) == P, "union of L and D must equal P")

  P.subsets.toSet // G \subseteq P
   .filter(G => (G intersect D).size <= k) // such that |G \cap D| <= k
   .argMax(G => G.size + G.uniquepairs.build(Function.tupled(like)).size)
   .get
}

val (p1, p2, p3, p4) = (Person("p1"), Person("p2"), Person("p3"), Person("p4"))

val P = Set(p1, p2, p3, p4)
val L = Set(p1, p2, p3)
val D = Set(p4)
val relations = Set(
  p1 like p2,
  p1 like p3,
  p2 like p3,
  p3 like p4
)
def like = relations.deriveFun
val k = 3

val out = si4(P, L, D, like, k)

println(h2("Input:"))
VegaRenderer.render(relations.deriveGraph(P))
println(s"k=$k")

println(h2("Output:"))
println(out)

```
{% endscalafiddle %}

# Selecting Invitees V5

<span style="font-variant: small-caps;">Selecting invitees (version 5)</span>

*Input:* A set $$P$$, subsets $$L \subseteq P$$ and $$D \subseteq P$$ with $$L \cap D = \emptyset$$ and $$L \cup D = P$$, and a function $$like: P \times P \rightarrow \{true, false\}$$.

<span>*Output:* $$G \subseteq P$$ such that $$|G\cap L| + |X| + |G|$$ is maximized (where $$X = \{p_i,p_j \in G\}~|~like(p_i,p_j) = true  \wedge i\neq \}$$).</span>


{% scalafiddle template="Persons", layout="v40" %}
```scala
def si5(P: Set[Person],
        L: Set[Person],
        D: Set[Person],
        like: (Person, Person) => Boolean): Set[Person] = {
  requirement(L subsetOf P, "l must be a subset of p")
  requirement(D subsetOf P, "d must be a subset of p")
  requirement((L intersect D).isEmpty, "intersection between l and d must be emtpy")
  requirement((L union D) == P, "union of l and d must equal p")


  P.subsets.toSet // G \subseteq P
   .argMax(G => {
     (G intersect L).size // |G \cap L|
     + G.size // |G|
     + G.uniquepairs.build(pair => like(pair._1, pair._2)).size // |X|
   })
   .get
}

val (p1, p2, p3, p4) = (Person("p1"), Person("p2"), Person("p3"), Person("p4"))

val P = Set(p1, p2, p3, p4)
val L = Set(p1, p2, p3)
val D = Set(p4)
val relations = Set(
  p1 like p2,
  p1 like p3,
  p2 like p3,
  p3 like p4
)
def like = relations.deriveFun
val k = 3

val out = si5(P, L, D, like)

println(h2("Input:"))
VegaRenderer.render(relations.deriveGraph(P))
println(s"k=$k")

println(h2("Output:"))
println(out)
```
{% endscalafiddle %}

# Selecting Invitees V6

<span style="font-variant: small-caps;">Selecting invitees (version 6)</span>

*Input:* A set $$P$$, subsets $$L \subseteq P$$ and $$D \subseteq P$$ with $$L \cap D = \emptyset$$ and $$L \cup D = P$$, a function $$like: P \times P \rightarrow \{true, false\}$$, and a threshold value $$k$$.

<span>*Output:* $$G \subseteq P$$ such that $$|Y| \leq k$$ and  $$|G\cap L|+|G|$$ is maximized (where $$Y = \{p_i,p_j \in G\}~|~like(p_i,p_j) = false \wedge i\neq j \}$$).</span>

{% scalafiddle template="Persons", layout="v40" %}
```scala
def si6(P: Set[Person],
        L: Set[Person],
        D: Set[Person],
        like: (Person, Person) => Boolean,
        k: Int): Set[Person] = {
    requirement(L subsetOf P, "l must be a subset of p")
    requirement(D subsetOf P, "d must be a subset of p")
    requirement((L intersect D).isEmpty, "intersection between l and d must be emtpy")
    requirement((L union D) == P, "union of l and d must equal p")


  P.subsets.toSet // G \subseteq P
   .filter(G => G.uniquepairs.build(pair => !like(pair._1, pair._2)).size <= k)
   .argMax(G => (G intersect L).size + G.size)
   .get
}

val (p1, p2, p3, p4) = (Person("p1"), Person("p2"), Person("p3"), Person("p4"))

val P = Set(p1, p2, p3, p4)
val L = Set(p1, p2, p3)
val D = Set(p4)
val relations = Set(
  p1 like p2,
  p1 like p3,
  p2 like p3,
  p3 like p4
)
def like = relations.deriveFun
val k = 3

val out = si6(P, L, D, like, k)

println(h2("Input:"))
VegaRenderer.render(relations.deriveGraph(P))
println(s"k=$k")

println(h2("Output:"))
println(out)
```
{% endscalafiddle %}
