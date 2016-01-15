---
layout: post
section-type: post
title: Identifying Probability Distributions
category: statistics, probability
tags: [ 'concept']
---

The following is the definition of probability distribution taken straight from [wikipedia](https://en.wikipedia.org/wiki/Probability_distribution).

In probability and statistics, a probability distribution assigns a probability to each measurable subset of the possible outcomes of a random experiment, survey, or procedure of statistical inference. Examples are found in experiments whose sample space is non-numerical, where the distribution would be a categorical distribution; experiments whose sample space is encoded by discrete random variables, where the distribution can be specified by a probability mass function; and experiments with sample spaces encoded by continuous random variables, where the distribution can be specified by a probability density function. More complex experiments, such as those involving stochastic processes defined in continuous time, may demand the use of more general probability measures.

The following is a rule of thumb to be applied for modelling distributions in continous and discrete case.

### Discrete

#### Bernoulli
We apply Bernoulli in the case where we want to model one occurrence of a success or failure trail. For example a coin-flip experiment can be thought of a Bernoulli Experiment where there are only 2 outcomes, heads (aka success) or tails(aka failure). 

#### Binomial
We apply Binomial where we model a number of success runs out of the total number of n runs, each with a probability of success p. 
Say suppose components are packed in boxes of 100. The probability of a component being defective is 0.2. Say if we want to find the probability of 5 components being defective we use Binomial Distribution for it.


#### Poisson
With Poission, we model the number of events that occur in a fixed interval. These events occur at some average rate independently of the previous events.
We model traffic events using Poisson

#### Geometric
Geometric is nothing but sequence of Bernoulli trials until first success (p)

### Continous

#### Uniform
Uniform is used in case where any of the values in the interval of a to b are equally likely

#### Gaussian
Commonly occurring distribution shaped like a bell curve. This often comes up because of the Central Limit Theorem (to be discussed later)

#### Exponential
We model time between Poisson events, where these events occur continously and independently.

Adios!
