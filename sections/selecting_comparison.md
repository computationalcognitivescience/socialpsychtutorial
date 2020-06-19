---
layout: index
title: Simulations for Dialogue 1 - Comparing model behavior
permalink: /dialogue1_comparison/
sidebar_link: false
sidebar_sort_order: 99
---

<div id="toc-wrapper" markdown="1">
* Table of contents
{:toc}
</div>

We take two approaches to analyze differences in model behavior. First, we
explore model behavior by hand. Second, we explore it by large scale simulation.

# Comparing model behaviour by hand
If you were already comparing the three models on the previous page, you
may have noticed differences in behavior. If not, you can use the following
code box to try different groups and see what selection of invitees each
model makes.

{% scalafiddle template="PersonsWithFormalisations" %}
```scala
val (p1, p2, p3, p4, p5) = (Person("p1"), Person("p2"), Person("p3"), Person("p4"), Person("p5"))

val P = Set(p1, p2, p3, p4, p5)
val L = Set(p1, p2, p3)
val D = Set(p4, p5)
val relations = Set(
  p1 like p2,
  p1 like p3,
  p2 like p3,
  p3 like p4,
  p1 like p5,
  p2 like p5
)
def like = relations.deriveFun
val k = 3

val out4 = si4(P, L, D, like, k)
val out5 = si5(P, L, D, like)
val out6 = si6(P, L, D, like, k)

println(h2("Input:"))
VegaRenderer.render(relations.deriveGraph(P))
println(s"k=$k (when applicable)")

println(h2("Output:"))
println(s"SI4 selects: $out4")
println(s"SI5 selects: $out5")
println(s"SI6 selects: $out6")
```
{% endscalafiddle %}

# Comparing model behaviour large scale

In the simulation experiment below, we randomly create groups of people and
their relationships to each other and the host. We can do this ```sampleSize```
times. For each group of friends generated, we ask three agents (corresponding
to the three models SI4, SI5 and SI6) to select invitees, resulting in three
(possibly different) outputs.

We then perform some data analysis computing a property of the input graph $$P$$,
viz. the ratio of likes and dislikes; and computing two example dependent variables:

1. The average number of pairs that like each other amongst invitees; and
2. The number of invited guests.

Try to play around with the parameters ```groupSize``` and ```sampleSize```
and see what changes. For example, increasing the number of samples, decreases
the variation in the data.

{% scalafiddle template="PersonsWithFormalisations" %}
```scala
val groupSize = 5
val sampleSize = 50

val P = List.tabulate(groupSize)(_ => Person.random).toSet

val results = for(trialNr <- 0 until sampleSize) yield {
  // Generate random relationships with the host
  val ld = P.toList.splitAt(Random.nextInt(P.size))
  val L = ld._1.toSet
  val D = ld._2.toSet
  // Generate random relations between pairs of people
  val relations = P.uniquepairs
    .map(pair => if(Random.nextBoolean) pair._1 like pair._2 else pair._1 dislike pair._2)

  // Three agents select invitees
  val outputSI4 = si4(P, L, D, relations.deriveFun, 1)
  val outputSI5 = si5(P, L, D, relations.deriveFun)
  val outputSI6 = si6(P, L, D, relations.deriveFun, 2)

  // Compute independent variables
  val nrLikes = relations.count(_.liking)
  val nrDislikes = relations.count(!_.liking)
  val ldRatio = nrLikes.toDouble / nrDislikes

  // Compute dependend variables
  val partySize4 = outputSI4.size
  val partySize5 = outputSI5.size
  val partySize6 = outputSI6.size

  val like4 = for(g1 <- outputSI4.toList) yield
      (for(g2 <- outputSI4.toList if(g1!=g2)) yield relations.contains(g1 like g2)).count(_ == true)  
  val like5 = for(g1 <- outputSI5.toList) yield
      (for(g2 <- outputSI5.toList if(g1!=g2)) yield relations.contains(g1 like g2)).count(_ == true)
  val like6 = for(g1 <- outputSI6.toList) yield
      (for(g2 <- outputSI6.toList if(g1!=g2)) yield relations.contains(g1 like g2)).count(_ == true)

  val avgLike4 = like4.sum.toDouble / like4.length
  val avgLike5 = like5.sum.toDouble / like5.length
  val avgLike6 = like6.sum.toDouble / like6.length

  // Return the dataset for this random graph
  (Map("likes" -> nrLikes, "dislikes" -> nrDislikes, "ldRatio" -> ldRatio, "avgLike" -> avgLike4, "partySize" -> partySize4),
  Map("likes" -> nrLikes, "dislikes" -> nrDislikes, "ldRatio" -> ldRatio, "avgLike" -> avgLike5, "partySize" -> partySize5),
  Map("likes" -> nrLikes, "dislikes" -> nrDislikes, "ldRatio" -> ldRatio, "avgLike" -> avgLike6, "partySize" -> partySize6))
}

val si4Data = results.map(_._1).toList
val si5Data = results.map(_._2).toList
val si6Data = results.map(_._3).toList

render(traces = List(Trace("SI4", si4Data), Trace("SI5", si5Data), Trace("SI6", si6Data)),
      xValue = "ldRatio",
      xLabel = "Likes / dislikes ratio in P",
      yValue = "avgLike",
      yLabel = "Average likes amongst invitees",
      title = "Like/Dislike ratios",
      plotType = PlotType.Point)

render(traces = List(Trace("SI4", si4Data), Trace("SI5", si5Data), Trace("SI6", si6Data)),
      xValue = "ldRatio",
      xLabel = "Likes / dislikes ratio in P",
      yValue = "partySize",
      yLabel = "Number of invitees",
      title = "Party size",
      plotType = PlotType.Point)

```
{% endscalafiddle %}


|Previous|Next|
|:--- | ---:|
|[Simulations for Dialogue 1](/dialogue1) |[Conclusions](/dialogue1_conclusions)|
