---
layout: post
title: Kotlin Spelling Correct
category: kotlin
tags: kotlin
published: true
summary: Kotlin spelling correct example
---

Converted from [Novig's spell-correct](http://norvig.com/spell-correct.html)

Mostly Kotlin with some assistance from Guava library for HashMultiset, Splitter and CharMatcher.

~~~java

class Correction(var resource : String) {

    var alphabet = "abcdefghijklmnopqrstuvwxyz"

    var wordsN = train(words())

    fun List<String>.or(a: List<String>): List<String> {
        return if (this.isNotEmpty()) this else a
    }

    fun loadResource(): URL {
        return this.javaClass.getResource(resource)
    }

    fun words(): String {
        return loadResource().readText(charset = Charsets.ISO_8859_1)
    }

    fun train(words: String): HashMultiset<String> {
        val alphas = Splitter.on(CharMatcher.WHITESPACE)
            .trimResults(CharMatcher.inRange('a', 'z').negate())
        return HashMultiset.create(alphas.split(words))
    }

    fun edits1(word: String): Set<String> {
        
        var splits = IntRange(0, word.length()).map { it -> Pair(word.take(it), word.drop(it)) }
        
        var edits1 = hashSetOf<String>()
        
        splits.filter { it -> it.second.isNotEmpty() }
            .mapTo(edits1) { it -> it.first.concat(it.second.substring(1)) }
        
        splits.filter { it -> it.second.length() > 1 }
            .mapTo(edits1) {
                it -> it.first + it.second.get(1) + it.second.get(0) + it.second.substring(2)
            }
        
        alphabet.flatMapTo(edits1) { alpha -> splits filter { it.second.isNotEmpty() } 
            map { it -> it.first + alpha + it.second.substring(1) } 
        }
        
        alphabet.flatMapTo(edits1) { alpha -> splits map { 
            it -> it.first + alpha + it.second }
        }
        
        return edits1
    }

    fun known_edits2(word: String): List<String> {
        return edits1(word).flatMapTo(arrayListOf<String>()) { e1 -> edits1(e1) filter {
        e2 -> wordsN.contains(e2) } map { e2 -> e2 } }
    }

    fun known(words: List<String>): List<String> {
        return words filter { word -> wordsN.contains(word) }
    }

    fun correct(word: String): String {
        var candidates = listOf(word)
        candidates = known(candidates) or known(edits1(word).toList()) or known_edits2(word) or candidates
        return candidates.maxBy { wordsN.count(it) }.orEmpty()
    }

}

fun main(args: Array<String>) {
    println(Correction("/big.txt").correct("transparen")) //transparent
}

~~~