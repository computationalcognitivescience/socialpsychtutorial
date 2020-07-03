---
layout: index
title: Dialogue 4 - Group formation
permalink: /dialogue4/
sidebar_link: true
sidebar_sort_order: 5
---

<div id="toc-wrapper" markdown="1">
* Table of contents
{:toc}
</div>

On this page you can find implementations of variants 1 and 2 of
<span style="font-variant: small-caps;">Party Subgrouping</span>. Included
are the formalizations themselves and the code to run a simulation for them.
You need to press the run button to run the simulation. This then also allows
you to change the input of the simulation to explore the behavior of the
models. You are encouraged to explore the simulations to your heart's content.
Afterwards, you can [compare the models through analysis here](/socialpsychtutorial/dialogue4_comparison).


# Party Subgrouping V1

<span style="font-variant: small-caps;">Party Subgrouping (version 1)</span>

*Input:* A set of guests $$G$$ and a function $$sim: G \times G \rightarrow \mathbb{R}$$.

*Output:* A partition of $$G$$ into non-overlapping subsets $$G_1, G_2, ..., G_k$$ that maximizes average ingroup similarity:
$$\frac{1}{k}\sum_{i=1,2,\dots k}sim(G_i) $$

Where ingroup similarity for subset $$G_i$$ is defined as mean pair-wise similarity:
$$sim(G_i)=\frac{1}{|G_i|}\sum_{g_i, g_j \in G_i} sim(g_i, g_j)$$

{% scalafiddle template="Persons", minheight="1000", layout="v45" %}
```scala
def ps1(G: Set[Person],
        sim: (Person, Person) => Double): Option[Set[Set[Person]]] = {
  def inGroupSim(subgroup: Set[Person]): Double =
    subgroup.uniquepairs.map(Function.tupled(sim)).sum / subgroup.size.toDouble

  G.allPartitionings
   .argMax(partitioning => {
     partitioning.map(Gi => inGroupSim(Gi) / partitioning.size).sum
   })
}

val a = Person("A")
val b = Person("B")
val c = Person("C")
val d = Person("D")
val e = Person("E")
val G = Set(a, b, c, d, e)
val similarities = Set(
  Similarity(a, b, 1.0),
  Similarity(a, c, 2.0),
  Similarity(a, d, -1.0),
  Similarity(a, e, 3.5),
  Similarity(b, c, -2.0),
  Similarity(b, d, 4.0),
  Similarity(c, d, -3.0)
)
def sim = similarities.deriveFun

val out = ps1(G, sim)

println(h2("Input:"))
VegaRenderer.render(similarities.deriveGraph(G))

println(h2("Output:"))
println(out)

```
{% endscalafiddle %}

# Party Subgrouping V2

<span style="font-variant: small-caps;">Party Subgrouping (version 2)</span>

*Input:* A set of guests $$G$$, a function $$sim: G \times G \rightarrow \mathbb{R}$$, and threshold of satisfactory similarity $$s \in \mathbb{R}$$.

*Output:* A partition of $G$ into non-overlapping subsets $$G_1, G_2, ..., G_k$$ where each partition has satisfactory ingroup similarity:
$$\forall_{i=1,2,\dots k}\left[sim(G_i) \geq s\right]$$

Where ingroup similarity for subset $G_i$ is defined as mean pair-wise similarity:
$$sim(G_i)=\frac{1}{|G_i|}\sum_{g_i, g_j \in G_i} sim(g_i, g_j)$$

{% scalafiddle template="Persons", minheight="1000", layout="v45" %}
```scala
def ps2(G: Set[Person],
        sim: (Person, Person) => Double,
        s: Double): Option[Set[Set[Person]]] = {
  def inGroupSim(subgroup: Set[Person]): Double =
    subgroup.uniquepairs.map(Function.tupled(sim)).sum / subgroup.size.toDouble

  G.allPartitionings
   .filter(_.forall(Gi => inGroupSim(Gi) >= s))
   .random
}

val a = Person("A")
val b = Person("B")
val c = Person("C")
val d = Person("D")
val e = Person("E")
val G = Set(a, b, c, d, e)
val similarities = Set(
  Similarity(a, b, 1.0),
  Similarity(a, c, 2.0),
  Similarity(a, d, -1.0),
  Similarity(a, e, 3.5),
  Similarity(b, c, -2.0),
  Similarity(b, d, 4.0),
  Similarity(c, d, -3.0)
)
def sim = similarities.deriveFun
val s = 2.5 // If you get no valid output, try lowering this value.

val out = ps2(G, sim, s)

println(h2("Input:"))
VegaRenderer.render(similarities.deriveGraph(G))
println(s"s=$s")

println(h2("Output:"))
println(out)

```
{% endscalafiddle %}
