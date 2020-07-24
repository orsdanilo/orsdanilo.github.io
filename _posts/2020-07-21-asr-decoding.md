---
title: Decoders for Automatic Speech Recognition
last_modified_at: 2020-07-2eT23:45:00-03:00
mathjax: True
classes: wide
header:
  image: /assets/images/grammar_brcd.png
  teaser: /assets/images/grammar_brcd.png
categories:
  - Blog
tags:
  - ASR
  - Decoder
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

Decoding is the process of calculating which sequence of words is most likely to match the acoustic signal represented by the feature vectors [[1]](#references) of a given utterance (a continuous piece of speech beginning and ending with a clear pause). Decoding is executed based on three sources of information:

* **Acoustic model:** An ensemble of [Hidden Markov Models](https://en.wikipedia.org/wiki/Hidden_Markov_model) representing words or phonemes;
* **Language model:** A list of word sequence probabilities;
* **Lexicon:** A dictionary of words and their respective phonemes.

During decoding, we essentially use these to try to predict the word sequence $\hat{W}$ that best matches the acoustic observation $\boldsymbol{X}=X_1 X_2\ldots X_n$, obtained using Bayes rule:

$$
\begin{equation*}
\begin{split}
\boldsymbol{\hat{W}} &= arg\max_{\boldsymbol{W}}P(\boldsymbol{W}\mid \boldsymbol{X})\\
    &= arg\max_{\boldsymbol{W}}\frac{P(\boldsymbol{X}\mid \boldsymbol{W})P(\boldsymbol{W})}{P(\boldsymbol{X})}\\
    &\propto arg\max_{\boldsymbol{W}} (\underbrace{P(\boldsymbol{X} \mid \boldsymbol{W})}_{\substack{\text{Acoustic} \\ \text{model}}} \underbrace{P(\boldsymbol{W})}_{\substack{\text{Language} \\ \text{model}}})
\end{split}
\end{equation*}
$$


The acoustic analysis is performed by a Deep Neural Network, which is trained to output the posterior probability $P(\boldsymbol{X} \mid \boldsymbol{W})$ of obtaining this observation given a candidate word sequence. The prior probability $P(\boldsymbol{W})$ of a word sequence is obtained from a language model, containing the allowed and most frequent occurrences for a given language. Therefore, prior probability and posterior likelihood are combined in order to output the best prediction with the available information.

One solution for the search of the best word sequence involves the use of Weighted Finite State Transducers (WFSTs). I wrote about it in [this post]({% post_url 2020-07-21-asr-wfst %}), if you feel up for the challenge.

## References

[1] [Gruhn, R. E., Minker, W., & Nakamura, S. (2011). Statistical Pronunciation Modeling for Non-Native Speech Processing. Springer Berlin Heidelberg.](https://books.google.com.br/books?hl=pt-BR&lr=&id=H_rGeqqaulYC&oi=fnd&pg=PR3&dq=Gruhn,+R.+E.,+Minker,+W.,+%26+Nakamura,+S.+(2011).+Statistical+Pronunciation+Modeling+for+Non-Native+Speech+Processing.+Springer+Berlin+Heidelberg.&ots=fvEiLQNOnn&sig=xEkxaP7JGRYzddwxUg-6GQMEHN8#v=onepage&q=Gruhn%2C%20R.%20E.%2C%20Minker%2C%20W.%2C%20%26%20Nakamura%2C%20S.%20(2011).%20Statistical%20Pronunciation%20Modeling%20for%20Non-Native%20Speech%20Processing.%20Springer%20Berlin%20Heidelberg.&f=false)


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