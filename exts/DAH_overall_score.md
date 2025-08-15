# Extension specification: DAH_overall_score

Version 2.0

Vendor: DAH

Dependencies:

```
NRS 2.0+
```

## Overview

This extensions is used to combine factor scores in to a single number, called
the overall score.

This extension requires subscores and their combine weight to be defined by
another extension (see ```DAH_factors``` for an example). 

Since NRS 3.0, the combine function is excluded from the core spec, so this
extension now works differently. After step 3.2.3 of score calculation, every
entry $e$ is assigned two embedded score vectors $S_+(e)$ and $S_-(e)$.

Separately unembed these two vectors to have two unembedded score vectors
$S^*_+(e)$ and $S^*_-(e)$. Let $\mathcal{S}$ be the set of all subscores,
where each subscore $s \in \mathcal{S}$ is a set of factor scores and has
combine weight $w_s$, then:
$$ \operatorname{scalarize}(v) = \sum_{s \in \mathcal{S}} \left(\sum_{f \in s} v_f^{1/w_s}\right)^{w_s}. $$

Then, the overall scalar score is simply defined as:
$$ s(e) = \operatorname{scalarize}(S^*_+(e)) - \operatorname{scalarize}(S^*_-(e)) $$

## See Also

Extension ```DAH_anime_normalize``` for a way to transform the overall score to
a normalized score that suits common standards (5 is average, 6 is fine, etc.)
