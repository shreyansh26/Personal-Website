---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Solving Substitution Ciphers using Markov Chain Monte Carlo (MCMC)"
subtitle: "Deciphering substitution ciphers can be framed as a Markov chain problem and a simple Monte Carlo sampling approach can help solve them very efficiently"
summary: ""
authors: ["Shreyansh Singh"]
tags: [sampling, cryptography, probability, mcmc]
categories: [Mathematics, Machine Learning]
date: 2023-07-22T20:30:42+05:30
lastmod: 2023-07-22T20:30:42+05:30
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: true

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

I was reading about Markov Chain Monte Carlo (MCMC) recently and discovered a very famous application of using them to decrypt substitution ciphers. This blog is meant to serve as notes on how the problem can be framed as a Markov chain and how a simple yet smart Monte Carlo sampling approach can help solve it very efficiently. We won't be explaining what Markov process and MCMC is, however I'll add some references for that at the end of this post.

### Assumptions

We assume a very simple setting in which we only deal with the 26 lowercase alphabets of the English language and the space character. This can be extended to more characters, but that isn't imperative to understand how the solution works. It can be extended for a larger set of characters with a bit of effort and without changing the core algorithm.

### Solution

The input we have is just the ciphertext from an English plaintext. How do we go from the ciphertext to the plaintext using MCMC?

**Scoring Function**

We need a scoring function to check how good our current state is. The state here the current mapping we have for the alphabets (for the substitution cipher).

The scoring function can be defined in multiple ways. One option can be to check fraction of decrypted words that appear in an English dictionary. Another option is to define the score to be the probability of seeing the given sequence of decrypted characters, which can be calculated as the product of consecutive bigram probabilities, using the pair-wise probabilities based on an existing long piece of English text. 

In my implementation, I have used the second option. So, if the plaintext/ciphertext is say, "good life is...", the score will be the product of probabilities - $Pr[(g,o)] * Pr[(o,o)] * Pr[(o,d)] * Pr[(d, \<space\>)] * ...$ and so on. Here $Pr[(g,o)]$ is the estimate of the probability that "$o$" follows "$g$" in typical English text.

**Transition Function**

Once we have the scoring function, the transition can be defined as follows - 

> **Algorithm**   
Repeat for N iterations  
$\quad$Take current mapping $m$  
$\quad$Switch the image of 2 random symbols to produce mapping $m'$  
$\quad$Compute the score for $m'$ and $m$  
$\quad$If score for $m'$ is higher than score for $m$  
$\qquad$Accept  
$\quad$If not, then with a small probability   
$\qquad$Accept   
$\quad$Else  
$\qquad$Reject

This transition function ensures that the stationary distribution of the chain puts much higher weight on better mappings.

This is how it would look when translated into code - 
{{< gist shreyansh26 9e117289ae36e1b353581672335466b9 >}}

The above chain runs extremely fast and solves the cipher in a couple thousand iterations. 


The entire code for this project including the bigram frequency/probability calculation and the MCMC-based solver is here on Github.

### Additional references on Markov Chains/MCMC
* [Markov Chain Monte Carlo Without all the Bullshit](https://jeremykun.com/2015/04/06/markov-chain-monte-carlo-without-all-the-bullshit/)
* [How would you explain Markov Chain Monte Carlo (MCMC) to a layperson?](https://stats.stackexchange.com/questions/165/how-would-you-explain-markov-chain-monte-carlo-mcmc-to-a-layperson)