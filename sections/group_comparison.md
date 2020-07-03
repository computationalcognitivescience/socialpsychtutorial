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
If you were already comparing the two models on the previous page, you
may have noticed differences in behavior. If not, you can use the following
code box to try different groups and see what selection of invitees each
model makes.

{% scalafiddle template="PersonsWithFormalisations", minheight="1000", layout="v40" %}
```scala
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
val s = 0.2

val out1 = ps1(G, sim)
val out2 = ps2(G, sim, s)

println(h2("Input:"))
VegaRenderer.render(similarities.deriveGraph(G))
println(s"s=$s")

println(h2("Output:"))
println(s"PS1 forms: $out1")
println(s"PS2 forms: $out2")
```
{% endscalafiddle %}

# Comparing model behaviour large scale

In the simulation experiment below, we randomly create groups of people and
their similarity relationships. For each group of friends generated, we
apply the two party subgrouping models resulting in two (possibly different)
outputs.

We then perform some data analysis by computing a property of the input,
viz. the mean similarity between all pairs; and by computing two example dependent
variables:

1. The mean ingroup similarity of the formed groups;
2. The number of groups that are formed.

Errorbars report 96% confidence intervals. *Note: Group size will require exponentially
more computation time, don't try large values ($$>7$$) unless you have time
until the end of the universe.*

{% scalafiddle template="PersonsWithFormalisations" %}
```scala
val groupSize = 6
val sampleSize = 100
val k = 0.6
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
  val nrGroups1 = if(outputPS1.isDefined) outputPS1.get.size else "NA"
  val nrGroups2 = if(outputPS2.isDefined) outputPS2.get.size else "NA"


  // Return the dataset for this random graph
  (Map("meanSimilarity" -> meanSimilarity, "nrGroups" -> nrGroups1, "meanInGroupSimilarity" -> meanInGroupSimilarity1),
  Map("meanSimilarity" -> meanSimilarity, "nrGroups" -> nrGroups2, "meanInGroupSimilarity" -> meanInGroupSimilarity2))
}

val ps1Data = results.map(_._1).toList
val ps2Data = results.map(_._2).toList

render(traces = List(Trace("PS1", ps1Data), Trace("PS2", ps2Data)),
      xValue = "meanSimilarity",
      xLabel = "Mean similarity in whole group",
      yValue = "meanInGroupSimilarity",
      yLabel = "Mean ingroup similarity",
      title = "Mean ingroup similarity",
      plotType = PlotType.Point)


render(traces = List(Trace("PS1", ps1Data), Trace("PS2", ps2Data)),
      xValue = "meanSimilarity",
      xLabel = "Mean similarity in whole group",
      yValue = "nrGroups",
      yLabel = "Number of groups",
      title = "Mean number of groups",
      plotType = PlotType.Point,
      true)

render(traces = List(Trace("PS1", ps1Data), Trace("PS2", ps2Data)),
      xValue = "nrGroups",
      xLabel = "Number of groups",
      yValue = "meanInGroupSimilarity",
      yLabel = "Mean ingroup similarity",
      title = "Number of groups vs ingroup similarity",
      plotType = PlotType.Point)

```
{% endscalafiddle %}
