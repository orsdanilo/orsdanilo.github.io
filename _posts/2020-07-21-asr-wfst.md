---
title: Automatic Speech Recognition
mathjax: True
classes: wide
header:
  image: /assets/images/hclg.png
  teaser: /assets/images/hclg.png
categories:
  - Blog
tags:
  - ASR
  - WFST
---

<script type="text/x-mathjax-config">
    MathJax.Hub.Config({
      tex2jax: {
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
        inlineMath: [['$','$']]
      }
    });
  </script>
  <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

<style>
table:nth-of-type(1) {
    display:table;
    width:100%;
}
</style>


Automatic Speech Recognition (ASR) systems convert speech from a recorded audio signal to text. An ASR system aims to infer the original words given an observable signal, most commonly following a probabilistic approach [[1]](#references). 

![ASRtest](/assets/images/ASRtest.png)  
**Figure 1:** Diagram of an ASR system.
{: style="font-size: 80%; text-align: center;"}
{: #fig1}

The input audio is split into overlapping frames of 25ms shifted by 10ms, so that within this tiny time window, the speech signal is considered to be stationary, allowing for the analysis featured here. Before decoding, these are fed to a feature extractor, which reduces signal dimensionality and extracts the most relevant information, that is, the linguistic message.

Decoding is the process of calculating which sequence of words is most likely to match the acoustic signal represented by the feature vectors [[1]](#references). Decoding is executed based on three sources of information:

* **Acoustic model:** An ensemble of HMMs representing words or phonemes;
* **Language model:** A list of word sequence probabilities;
* **Lexicon:** A dictionary of words and their respective phonemes.

During decoding, we essentially use these to try to predict the word sequence $\hat{W}$ that best matches the acoustic observation $\boldsymbol{X}=X_1 X_2\ldots X_n$, as seen in

$$
\begin{equation*}
\begin{split}
\boldsymbol{\hat{W}} &= arg\max_{\boldsymbol{W}}P(\boldsymbol{W}\mid \boldsymbol{X})\\
    &= arg\max_{\boldsymbol{W}}\frac{P(\boldsymbol{X}\mid \boldsymbol{W})P(\boldsymbol{W})}{P(\boldsymbol{X})}\\
    &\propto arg\max_{\boldsymbol{W}} (\underbrace{P(\boldsymbol{X} \mid \boldsymbol{W})}_{\substack{\text{Acoustic} \\ \text{model}}} \underbrace{P(\boldsymbol{W})}_{\substack{\text{Language} \\ \text{model}}})
\end{split}
\end{equation*}
$$

The acoustic analysis is performed by a Deep Neural Network, which is trained to output the posterior probability of obtaining this observation given a candidate word sequence. The prior probability of a word sequence is obtained from a language model, containing the allowed and most frequent occurrences for a given language. Therefore, prior probability and posterior likelihood are combined in order to output the best prediction with the available information.

One solution for the search of the best word sequence involves the use of WFSTs, presented in the following section.
  
Weighted Finite State Transducers
----------

Weighted Finite State Transducers (WFSTs) are static graphs encoding the whole word-sequence search space. They contain many levels of information, going from the acoustic model to phones in context, to single phonemes, to words, and finally, words in sequence.

A finite-state transducer is a finite automaton whose state transitions are labeled with both input and output symbols. Therefore, a path through the transducer encodes a mapping from an input to an output string. They provide a common and natural representation for major components of speech recognition systems, including Hidden Markov Models (HMMs), context-dependency models, pronunciation dictionaries, statistical grammars and word or phone lattices [[2]](#references). The decoder goes through the WFST creating a lattice, a graph structure that contains the most probable - or less costly - paths for a given utterance (a continuous piece of speech beginning and ending with a clear pause), taking into account both acoustic and language costs.

A transducer $T=(\mathcal{A}, \mathcal{B}, \mathcal{Q}, I, F, E, \lambda, \rho)$ over a semiring $\mathbb{K}$ is characterized by:

* $\mathcal{A}$: finite input alphabet;
* $\mathcal{B}$: finite output alphabet;
* $\mathcal{Q}$: finite set of states;
* $I$: set of initial states;
* $F$: set of final states;
* $E$: finite set of transitions;
* $\lambda$: initial state weight assignment;
* $\rho$: final state weight assignment.

The set of transitions is $E\subseteq\bar{E}=\mathcal{Q}\times(\mathcal{A}\cup\{\epsilon\})\times(\mathcal{B}\cup\{\epsilon\})\times\mathbb{K}\times\mathcal{Q}$. This means it is a subset of all possibilities of transition from one state to another (or to itself) with an input label, an output label and a weight.

The operations are executed on specific *semirings*. A semiring is a set $\mathbb{R}$ equipped with two binary operators addition ($\oplus$) and multiplication ($\otimes$), with identity elements $\bar{0}$ and $\bar{1}$, respectively. One simple example of a semiring is the set of natural numbers (including zero) under ordinary addition and multiplication [[3]](#references). Two commonly used semirings in ASR are the *probability semiring* and the *log semiring*, shown in [Table 1](#tab1).

| SEMIRING | SET | $\oplus$ | $\otimes$ | $\bar{0}$ | $\bar{1}$ |
|:---:|:---:|:---:|:---:|:---:|:---:|
| Probability | $\mathbb{R}_+$ | $+$ | $\times$ | 0 | 1 |
| Log | $\mathbb{R} \cup \{-\infty,+\infty\}$ | $\oplus_{log}$ | + | $+\infty$ | 0 |
{: #tab1}
**Table 1:** Probability and log semirings. $\oplus_{log}$ is defined by: $x \oplus y=-log(e^{-x}+e^{-y})$.}
{: style="font-size: 80%; text-align: center;"}

There is a set of operations we can perform on transducers. The main ones are the following:

* **Composition:** Combining graphs, therefore different levels of representation;
* **Determinization:** Making an automaton deterministic, that is, each state should have at most one transition with any given input label, which should not be the empty ($\epsilon$) symbols;
* **Minimization:** Reducing the size of a deterministic automaton.

An usual Kaldi decoder contains a set of four WFSTs which are combined (or composed, as explained in later sections): HMM-level, Context Dependency, Pronunciation Lexicon and Grammar. The mapping from input to output across transducers is shown in [Figure 2](#fig2).

![hclg](/assets/images/hclg.png)  
**Figure 2:** Mapping of input and output across transducers.
{: style="font-size: 80%; text-align: center;"}
{: #fig2}

[Figure 3](#fig3) is the representation of a grammar with full sentences. Arcs show legal word strings and their probabilities. As input and output labels are the same, we call this automaton an acceptor (it ends at a final state if the sequence is accepted by the WFST). The graph in [Figure 4](#fig4) corresponds to a pronunciation lexicon, showing different possibilities of pronunciation and mapping a phone string to a word string in the lexicon. Since it *transduces* phones to words, it is called a transducer. Here are some Weighted finite-state transducer examples:

![grammar_brcd](/assets/images/grammar_brcd.png)  
**Figure 3:** Example grammar FST.
{: style="font-size: 80%; text-align: center;"}
{: #fig3}

![lexicon_brcd](/assets/images/lexicon_brcd.png)  
**Figure 4:** Example pronunciation lexicon FST.
{: style="font-size: 80%; text-align: center;"}
{: #fig4}

Essentially, the grammar graph shows us the allowed sequences of words along with the probability of their occurrence. It can be hand crafted through the use of regular expressions, rules or even direct build, but it can most notably be learned from data, creating stochastic $n$-gram models from text corpora.

The lexicon graph, on the other hand, presents the allowed pronunciations for the words. It is the Kleene closure of the union of individual word pronunciations, that is, the set of pronunciations for all word sequences. This means that there are loops in this automaton. For matters of efficiency, output word labels should be placed on the initial transition of words.

Below is a visual example of graph growth with composition of H, C, L and G transducers.

![lg_brcd](/assets/images/lg_brcd.png)  
**Figure 5:** Example $L\circ G$.
{: style="font-size: 80%; text-align: center;"}
{: #fig5}

![clg_brcd](/assets/images/clg_brcd.png)  
**Figure 6:** Example $C\circ L\circ G$.
{: style="font-size: 80%; text-align: center;"}
{: #fig6}

![hclg_brcd](/assets/images/hclg_brcd.png)  
**Figure 7:** Example $C\circ L\circ G$.
{: style="font-size: 80%; text-align: center;"}
{: #fig7}

The operation of composition essentially consists in checking transitions in both input transducers and adding an arc to the output graph whenever the output label of the first transducer matches the input of the second one. This creates a mapping between the input of $T_1$ and the output of $T_2$. A visual example, also from [[2]](#references), is shown in [Figure 8](#fig8).

![composition](/assets/images/composition.png)  
**Figure 8:** Example of a basic transducer composition [[2]](#references). The upper left transducer is $T_1$, the upper right one is $T_2$, and the graph below them is the result.
{: style="font-size: 80%; text-align: center;"}
{: #fig8}

During decoding, the decoder examines the word-sequence search space beginning with start states and evaluates transition costs in order to build a *lattice*, a representation of the alternative $n$-best word-sequences 

Below is an illustration of decoding operation. For each time frame, visited states are saved in the form of tokens, with links to other states. Notice how not all states and transitions make their way into the lattice (and those that are there can still be removed via pruning).

![lattice_decoded](/assets/images/static_searchspace.png)  
**Figure 9:** Example FST examined during decoding.
{: style="font-size: 80%; text-align: center;"}
{: #fig9}

![lattice_decoded](/assets/images/lattice_decoded.png)  
**Figure 10:** Example lattice created from decoding.
{: style="font-size: 80%; text-align: center;"}
{: fig10}

Kaldi creates a data structure corresponding to a full state-level lattice, so for every arc of $HCLG$ traversed, a separate arc is created in the lattice, with acoustic and graph cost stored separately, as illustrated in [Figure 10](#fig10). This structure is pruned periodically, and the final pruned graph then has its epsilons removed and is determinized [[4]](#references).

More on the theory of weighted finite state transducers can be found in [[2]](#references).

## References

[1] Gruhn, R. E., Minker, W., & Nakamura, S. (2011). Statistical Pronunciation Modeling for Non-Native Speech Processing. Springer Berlin Heidelberg.

[2] Mohri, M., Pereira, F., & Riley, M. (2008). Speech recognition with weighted finite-state transducers. In Springer Handbook of Speech Processing (pp. 559–584). Springer.

[3] Akian, M., Gaubert, S., & Guterman, A. (2009). Linear independence over tropical semirings and beyond. Contemporary Mathematics, 495, 1.

[4] Povey, D., Hannemann, M., Boulianne, G., Burget, L., Ghoshal, A., Janda, M., Karafiát, M., Kombrink, S., Motlı́ček Petr, Qian, Y., & others. (2012). Generating exact lattices in the WFST framework. Acoustics, Speech and Signal Processing (ICASSP), 2012 IEEE International Conference On, 4213–4216.

{% if page.comments %}

<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://orsdanilo-github-io.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>

{% endif %}