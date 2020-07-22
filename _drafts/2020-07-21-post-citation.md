---
title: A Blogging Scholar
mathjax: True
---
[comment]: <> (<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>)

<script type="text/x-mathjax-config">
    MathJax.Hub.Config({
      tex2jax: {
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
        inlineMath: [['$','$']]
      }
    });
  </script>
  <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>


{{ page.title }}
================

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor
incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis
nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
Duis 'aute irure dolor in reprehenderit in voluptate'
velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat
cupidatat non proident, 'sunt in culpa qui officia deserunt mollit anim id est
laborum' {% cite ruby %}.

$$
\begin{equation*}
\begin{split}
\boldsymbol{\hat{W}} &= arg\max_{\boldsymbol{W}}P(\boldsymbol{W}\mid \boldsymbol{X})\\
    &= arg\max_{\boldsymbol{W}}\frac{P(\boldsymbol{X}\mid \boldsymbol{W})P(\boldsymbol{W})}{P(\boldsymbol{X})}\\
    &\propto arg\max_{\boldsymbol{W}} (\underbrace{P(\boldsymbol{X} \mid \boldsymbol{W})}_{\substack{\text{Acoustic} \\ \text{model}}} \underbrace{P(\boldsymbol{W})}_{\substack{\text{Language} \\ \text{model}}})
\end{split}
\end{equation*}
$$

Duis 'aute irure dolor in reprehenderit in voluptate' {% cite ruby %}
velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat
cupidatat non proident, 'sunt in culpa qui officia deserunt mollit anim id est
laborum' {% cite openfst %}.

* $\mathcal{A}$: finite input alphabet;
* $\mathcal{B}$: finite output alphabet;
* $\mathcal{Q}$: finite set of states;
* $I$: set of initial states;
* $F$: set of final states;
* $E$: finite set of transitions;
* $\lambda$: initial state weight assignment;
* $\rho$: final state weight assignment.


References
----------

{% bibliography --cited %}