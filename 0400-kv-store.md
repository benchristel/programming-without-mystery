# Case Study 1: An Immutable Key-Value Store

In this chapter, I'll be exploring the implementation of
an immutable key-value store with an HTTP API. Here,
*immutable* means that overwritten values in the store are
preserved, and can be accessed later by revision number.

The code for this project is in Ruby. Ruby is an interpreted
object-oriented language, vaguely similar to Smalltalk,
Python, and Perl. I've rewritten some of the example code
to avoid Rubyisms that would take a lot of explanation and
distract from the main point.

## Technique: Readme-Driven Development

## Technique: Start With a Walking Skeleton

## Technique: Scaffold the Project With a System Test

## Technique: Randomize Test Ordering to Expose Test Pollution

## Technique: Introduce a Naïve Unit-Testing Seam

## Technique: Separate Unit and System Tests

## Technique: Remove Duplication With the Flocking Rules

## Technique: Refactor to Symmetry

## Technique: Keep Your Options Open With Contract Tests

## Technique: Implement Contracts Cheaply at First

## Technique: Always Represent Requirements Changes In The Contract

## Technique: Comb Your Lexicon For Tiny Objects

e.g. Revision

## Technique: Reproduce Bugs With Tests

## Technique: Design Up Front

## Technique: Write an Inefficient Reference Implementation

Implement a rolling hash with bigints and expensive modular
arithmetic—then try to write a more efficient implementation.

## Technique: Testing When You Don't Know the Right Answer

How to test that the rolling hash is "good"? I.e. that it
finds chunk boundaries at appropriate intervals.
