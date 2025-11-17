# nrs

Unified personal media rating pipeline.

> [!NOTE]
> NRS stands for _New Rework System_ or _New Ranking System_. The name is quite
> old and is quite misleading now.

Originally developed to rate anime in 2021, NRS had become a general-purpose
media rating tool that can be used for movies, TV shows, books, music and more.
Each media item in NRS is called an **entry**.

NRS utilizes the same rating procedure for ALL entries, which means entries of
different media types can be rated and compared on the same scale. Three-minute
songs with higher rating than a full-length 12-episode anime series is a common
occurrence in the system.

## High-Level Overview

The main scoring procedure of NRS is specified in the core specification.
Basically, an entry can gain score from impacts (constant scores) or relations
(scores derived from other entries).

Scores are multidimensional, since different subscores are theoretically
unexchangable. There are extensions to collapse the score vector into a scalar
score, but this is a lossy and subjective operation.

To combine scores from different impacts/relations, NRS uses combine functions
to do so. It is provable that all _nice_ combine functions have the form of
$\ell_p$-norm, as illustrated in this
[blog](https://btmxh.github.io/blog/posts/combine/), so since NRS 3.0, this
particular family of combine functions is fixed in the core spec. Since the
$\ell_p$-norm basically "embed" a score into a powered-space (taking power),
summing up the scores there, before "unembedding" (taking root) it to the
original score space, this operation is called **embedding** in NRS.

Prior to NRS 3.0, circular relations are solved via iterative approximation.
However, this process can be made exact after fixing the combine function to
$\ell_p$-norm, so since NRS 3.0, circular relations are solved via a system of
linear equations. However, the "SCC and toposort" procedure is still used to
optimize the solving process.

## Markup

The complicated part of NRS is markup, which is defining entries and their
respective impacts and relations. There are plans to automate this via a unified
pipeline (fetching metadata, parsing album notes, etc.), but currently this is
still done manually via a markup language.

The NRSML markup language, available at the
[nrsml repo](https://github.com/nrs-org/nrsml)
is an opinionated markup language specifically designed for NRS with
requirements on some `DAH_` extensions. It is basically XML with a schema for
autocompletion and validation.
