# NRS (New Rating System) Specification

Version 3.0

## 1. Overview

NRS is a scoring system (although the name is "New Ranking System"), which tries
to be as powerful as possible.

It decides how scores are calculated for **entries**, and this score depends on
**impacts** and **relations**.

Personally, my implementation of NRS is used to rank anime, manga and other
similar content (not limited to Japanese origin, but still mostly weeb stuff).

Since NRS 2.0, the concept of extensions is introduced into NRS. This
effectively modularize NRS, and allow NRS to be more flexible for different
implementations. The most notable result is that it remove the need of any
"Japanese word" from this specification (except for the Overview and History
part, and some examples).

NRS 3.0 introduces a breaking change by forcing the usage of the
previously-named _pow_ combine function. It can be proven mathematically that
the only continuous **associative** combine functions are the signed extension
of the following forms of unsigned combine function:
$$ c(x_1, x_2, \ldots, x_n) = \left(\sum_{k=1}^n x_k^p\right)^{1/p}, $$
or
$$ c(x_1, x_2, \ldots, x_n) = \max_{1 \le k \le n} x_k, $$
with the latter being an approximate of the former as $p \to \infty$.
NRS 3.0 only allows the first form of combine function, with $p = \frac1w$ being
the inverse of the combine weight.
Exploiting the nature of this choice, NRS 3.0 removed the concept of the combine
function entirely, and replaced it with the concept of _score embedding_.

## 2. Fundamental concepts

### 2.1. NRS systems and NRS contexts

This set of files, containing the core specification and extensions specification
documents are hosted on a [Git](https://git-scm.com/) reposistory. The current
official reposistory is hosted on [GitHub](https://github.com), with the
reposistory URL being
[https://github.com/nrs-org/nrs.git](https://github.com/nrs-org/nrs.git).
This Git reposistory may be referred as "the official NRS reposistory".

An NRS system is an entity that is capable of executing certain tasks described
by an NRS specification. This is the specification for a *core* NRS system,
therefore any *core* NRS system must be an NRS system. By modifying this
document using a set of [extensions](#22-extensions), which each has its own
specification in the `exts` directory of the NRS specification git repository,
a modified NRS specification can be derived from this document, then used to
create a new NRS system. If the modification change how the system behave,
this new system is no longer a *core* NRS system.

An NRS context can be created from an NRS system, which may or may not exist
physically. NRS contexts act as an environment for the tasks to be executed.
Different contexts from the same system may have different extensions enabled,
therefore their processes may be incompatible. A *core* NRS context is created
from a *core* NRS system, and since there are no restrictions on extension
support for *core* system, *core* contexts also may not be incompatible with
each other. After the creation of an NRS context, the set of enabled extensions
can not be modified.

> Context incompability is defined to be the inablity to interfere with each
other's processes. (i.e. sharing data)

### 2.2. Extensions

An extension is a set of modification to this document.

All extensions must have a unique **name**. This name must be a string of ASCII
characters, which may be an alphanumeric character (A-Z, a-z, 0-9) or the
underscore character `'_'`. The use of any other characters is strictly
forbidden. The uniqueness of extension names must be case-insensitive ('Abc' and
'abc' must not be names for different extensions).

The entity that specify an extension is called the vendor of that extension,
and may have a name used to prefix their extension names using the
OpenGL-inspired extension name convention. This naming convention is defined as
such: the vendor name and the base extension must be written in
[SCREAMING_SNAKE_CASE](https://en.wiktionary.org/wiki/screaming_snake_case)
and [snake_case](https://en.wikipedia.org/wiki/Snake_case), respecitively;
then joined by the underscore separator character. This helps minimizing
extension naming collisions where different vendors implementing the same
functionalities with minor differences.

This name is then used to name the specification files of the extension,
which is in the `exts` directory of the official NRS reposistory.

Extensions may interact with each other. There are two types of interactions
defined in this document:

* dependence: Some extensions, in order to be enabled, require some other
extensions to be enabled.
* incompatible: Some extensions, in order to be enabled, require some other
extensions to be disabled.

### 2.3. Mathematical concepts

The [score calculation process](#32-score-calculation) depends on some basic
concepts of linear algebra: vectors, matrices, dot product, matrix
multiplication, etc. All of the vectors and matrices used in the process will
be simple vectors and matrices (ordered tuples of real numbers).

Score vectors and matrices are simple vectors and matrices with a restriction:
in a context, all score vectors must have the same number of dimensions $n$,
and all score matrices must be $n \times n$ non-negative square matrices.
This number $n$ is called the number of factor scores.

Factor scores are fancy indices to access elements of a score vector. The number
of factor scores are globally constant in a context, and every factor score can
be used to access elements of any score vectors.

Every factor score has a **factor score weight**, which is a real number in $(0,
\infty)$. The factor score weight is used to determine how much the factor
score combines. For example,

* A factor score weight of 1 means that combining factor scores is simply taking
  the sum.
* A factor score weight of 0 means that combining factor scores is simply taking
  the maximum of all factor scores, which means only the largest factor score
  matters. Note that a factor score weight of $0$ is not allowed, but one can
  approximate it by using a very small positive number.
* Factor score weights in between 0 and 1 means that combining factor scores
  gives a total score that is between the maximum and the sum of all factor
  scores, giving a balance between the two extremes.
* Factor score weights greater than 1 means that combining factor scores
  can give a total score that is greater than the sum of all factor scores, which
  might be unsuitable in most cases.

**Embedding** is defined as the process of transforming a non-negative score vector
$v$ to the score vector $v'$, defined as:
$$ v'_i = v_i^{1/w_i}, $$
where $w_i$ denotes the factor score weight of factor score $i$.

**Unembedding** is the reverse process of embedding, defined as:
$$ v_i = v_i^{\prime w_i}. $$

NRS score calculation goes as follows: first, all user-facing scores (impact
scores) are embedded into the linear scoring phase, where embedded score vectors
are added together to get a overall linear score vector for each entry.
The overall linear score vector is then unembedded to get the overall score vector
of each entry.

### 2.4. Context entities

Context entities are entities that belong to an NRS context. These contain:
entries, impacts, and relations.

#### 2.4.1. Entries

An **entry** is a context entity that is scored in the
[score calculation process](#32-score-calculation)

Prior to NRS 3.0, an entry can contain other entries, however, this relationship
is subsumed by the relation entities.

#### 2.4.2. Impacts

An **impact** is a context entity that's used to give constant score to entries.

An impact is defined using two properties:

* The impact contribution map, mapping entries to a score matrix, called the
**contributing weight**. This number determines how much of the impact
was a result of the entry.
* The impact score vector, which will be scaled to get the score added
each contributing entry.

#### 2.4.3 Relations

A **relation** is a context entity that's used to give score to entries based on
scores of other entries. The score added to entries will depend on scores of
other entries.

A relation is defined using two properties:

* The relation referencing map, mapping entries to a score matrix, called the
**relation score transform matrix**.

* The relation contribution map, mapping entries to a score matrix, called the
**contributing weight**. Similarly to the weight in impacts, it
determines how much of the relation affects the entry.

## 3. Required capabilities

An NRS system must be able to carry out two operations: creating contexts and
calculating entries' scores.

### 3.1. Creating contexts

An NRS context must be able to be created from a system. In this context
creation process, the set of all enabled extensions will be specified. This
extension set is not required to be accessible from the newly created context,
but must stay the same in the lifetime of the context. The definition of this
lifetime is implementation-defined.

### 3.2. Score calculation

The score calculation process is the process of assigning a score vector to each
entry $e$, denoted as $S(e)$. Denote the set of all entries, impacts and
relations as $E$, $I$ and $R$, respectively, the contributing weight of an entry
$e$ in an impact/relation $ir$ as $W(e, ir)$, the relation score transform
matrix of an entry $e$ in the relation $r$ is $T(e, r)$ and the impact score of
an impact $i$ as $s(i)$.

#### 3.2.1. Embedding impact scores

All impact scores are embedded via the method described in
[2.3](#23-mathematical-concepts). However, since impact scores might be
mixed-sign, the scores are embedded into two parts: the positive part
and the negative part:
$$
\begin{align*}
s_+ (i) &= \operatorname{embed}(\max\{0, s(i)\})\\
s_- (i) &= \operatorname{embed}(\min\{0, s(i)\})
\end{align*}
$$

#### 3.2.2. Calculating constant scores

Then, the impact (constant) scores of each entries can be calculated as follows:
$$
\begin{align*}
S^c_+(e) &= \sum_{i \in I} W(e, i) \max\{s(i), 0\},\\
S^c_-(e) &= \sum_{i \in I} W(e, i) \max\{-s(i), 0\}.
\end{align*}
$$

#### 3.2.3. Solving relational scores

In this step, the relation scores (and therefore the overall scores) of each
entry are calculated. The total score of an entry $e$ is defined as the sum of
the impact and relation scores:
$$
\begin{align*}
S_+(e) &= S^c_+(e) + S^r_+(e),\\
S_-(e) &= S^c_-(e) + S^r_-(e),
\end{align*}
$$
where $S^r_+(e), S^r_-(e)$ is the two signed relation scores of entry $e$, which
should satisfy the following relation
$$
\begin{align*}
S^r_+(e) &= \sum_{r \in R} W(e, r) \sum_{e' \in E} T(e', r) S_+(e'),\\
S^r_-(e) &= \sum_{r \in R} W(e, r) \sum_{e' \in E} T(e', r) S_-(e').
\end{align*}
$$

Combining the two sets of equations gives,
$$
\begin{align*}
S_+(e) &= S^c_+(e) + \sum_{r \in R} W(e, r) \sum_{e' \in E} T(e', r) S_+(e'),\\
S_-(e) &= S^c_-(e) + \sum_{r \in R} W(e, r) \sum_{e' \in E} T(e', r) S_-(e'),
\end{align*}
$$
which is simply a system of linear equations that can be solved analytically.

#### 3.2.4. Unembedding overall scores

The overall scores of each entry $e$ is then calculated by unembedding the
embedded scores:
$$ S(e) = \operatorname{unembed}(S_+(e) - S_-(e)) $$
