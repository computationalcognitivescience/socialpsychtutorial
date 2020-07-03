---
layout: index
title: Dialogue 4 - Comparing models
permalink: /dialogue4_comparison/
sidebar_link: true
sidebar_sort_order: 6
---

<div id="toc-wrapper" markdown="1">
* Table of contents
{:toc}
</div>

We take two approaches to analyze differences in model behavior. First, we
explore model behavior by hand. Second, we explore it by large scale simulation.

# Comparing model behavior by hand
If you were already comparing the three models on the previous page, you
may have noticed differences in behavior. If not, you can use the following
code box to try different groups and see what selection of invitees each
model makes.

{% scalafiddle template="PersonsWithFormalisations", minheight="1000", layout="v40" %}
```scala
val a = Person("A")
val b = Person("B")
val c = Person("C")
val d = Person("D")
val G = Set(a, b, c, d)
val similarities = Set(
  Similarity(a, b, 1.0),
  Similarity(a, c, 2.0),
  Similarity(a, d, -1.0),
  Similarity(b, c, -2.0),
  Similarity(b, d, 4.0),
  Similarity(c, d, -3.0)
)
def sim = similarities.deriveFun
val s = 0.2

val out1 = ps1(G, sim)
val out2 = ps2(G, sim, s)

println(h2("Input:"))
VegaRenderer.render(similarities.deriveGraph(G))
println(s"s=$s")

println(h2("Output:"))
println(s"SI4 selects: $out1")
println(s"SI4 selects: $out2")
```
{% endscalafiddle %}

# Comparing model behaviour large scale

In the simulation experiment below, we randomly create groups of people and
their similarity relationships.

*Note: Group size will require exponentially
more computation time, don't try large values ($$>7$$) unless you have time
until the end of the universe.*

{% scalafiddle template="PersonsWithFormalisations" %}
```scala
val groupSize = 5
val sampleSize = 50
val k = .2
val minSimilarity = -4
val maxSimilarity = 4

val P = List.tabulate(groupSize)(_ => Person.random).toSet

val results = for(trialNr <- 0 until sampleSize) yield {
  // Generate random similarities between pairs of people
  val similarities = P.uniquepairs
    .map(pair => Similarity(
      pair._1,
      pair._2,
      Random.nextDouble() * (math.abs(minSimilarity) + maxSimilarity) + minSimilarity
    ))
  val sim = similarities.deriveFun

  // Three agents select invitees
  val outputPS1 = ps1(P, sim)
  val outputPS2 = ps2(P, sim, k)

  // Compute independent variables
  val meanSimilarity = similarities.map(_.degree).sum / P.size

  // Compute dependend variables
  val meanInGroupSimilarity1: Any = if(outputPS1.isDefined) {
    val partitioning = outputPS1.get
    partitioning.map(part => part.uniquepairs.map(Function.tupled(sim)).sum / part.size).sum / partitioning.size
  } else {
    "NA"
  }
  val meanInGroupSimilarity2: Any = if(outputPS2.isDefined) {
    val partitioning = outputPS2.get
    partitioning.map(part => part.uniquepairs.map(Function.tupled(sim)).sum / part.size).sum / partitioning.size
  } else {
    "NA"
  }

  // Return the dataset for this random graph
  (Map("meanSimilarity" -> meanSimilarity,"meanInGroupSimilarity" -> meanInGroupSimilarity1),
  Map("meanSimilarity" -> meanSimilarity, "meanInGroupSimilarity" -> meanInGroupSimilarity2))
}

val ps1Data = results.map(_._1).toList
val ps2Data = results.map(_._2).toList

render(traces = List(Trace("PS1", ps1Data), Trace("PS2", ps2Data)),
      xValue = "meanSimilarity",
      xLabel = "Mean similarity in whole group",
      yValue = "meanInGroupSimilarity",
      yLabel = "Mean ingroup similarity",
      title = "Mean ingroup similarity",
      plotType = PlotType.Bar,
      bin = true)

```
{% endscalafiddle %}
