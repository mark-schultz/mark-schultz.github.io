---
layout: post
title: Sensitivity as the Operator Norm of a Derivative
tags: differential-privacy weird-construction
comments: true
---

Differential Privacy is an area which has seen wide interest
(both in academia and real-life) in recent years.
But what *is* differential privacy?
As I don't work in this area, I'm rather ill-suited to say anything --- but
at least why does it have that name?
Frank McSherry (one of the founders of the area, who shares the [Godel
Prize](https://en.wikipedia.org/wiki/G%C3%B6del_Prize) as a result)
[states that it was one of several names
considered](https://crypto.stackexchange.com/questions/77324/what-does-the-term-differential-in-differential-privacy-mean), including others like "Marginal Privacy" or "Incremental Privacy".
He further states that he thinks the name "stuck" due to an analogy with
differential cryptanalysis.

This post is a relatively quick note on how "differential" objects show up
(implicitly) in differential privacy.
This is somewhat unsurprising, but formalizing this has been sufficiently weird
journey that it seemed like it should be shared with others.


# What is Differential Privacy?

To avoid this post dragging on too long, I'm going to answer "I'm not telling".
My plan is to do this *regardless if I really know*, so this response is actually
differentially private (ha).

For people who actually want to know, there are a variety of excellent technical
resources to learn.
In particular [The Algorithmic Foundations of Differential
Privacy](https://privacytools.seas.harvard.edu/files/privacytools/files/the_algorithmic_foundations_of_differential_privacy_0.pdf) by Dwork and Roth is great.
I'll defer to them for an introduction, and even adopt their notation.
Really, the entirety of this post is pointing out that a common parameter within
differential privacy can be interpreted in terms of a deceptively simple
equation.
To cut to the chase:

$$\Delta_{\vec x, p}(f) = \lVert \nabla f(\vec x)\rVert_p$$

Where $$\Delta_{\vec x, p}(f)$$ is the local $$\ell_p$$ sensitivity of $$f$$ at
$$\vec x$$.
This immediately implies the following for the global sensitivity
$$\Delta_p(f)$$:

$$\Delta_p(f) = \max_{\vec x\in\mathbb{N}^{|\mathcal{X}|}} \lVert \nabla f(\vec
x)\rVert_p$$

So in effect, if you define triangles correctly, you can "flip" one upside
down (or right-side up --- I do not know the natural orientation of triangles).
Before discussing the "discrete derivative" definitions which will be needed to
interpret the right-hand side, we should probably talk about the quantity on the
left-hand side.

# The sensitivity of a function

As mentioned before, I will use the notation of Dwork and Roth's book.
Definition 3.1 and 3.8 define the (global) $$\ell_1$$ and $$\ell_2$$ sensitivities.
Upon reading both of them, there is a straightforward generalization to
$$\ell_p$$ sensitivities, which I give below:

> The global $$\ell_p$$ sensitivity of a function $$f :
> \mathbb{N}^{|\mathcal{X}|}\to\mathbb{R}^k$$ is:
>
> $$\Delta_p(f) = \max_{\vec{x}, \vec{y}\in\mathbb{N}^{|\mathcal{X}|} : \lVert \vec x - \vec y \rVert_1 = 1} \lVert f(\vec x) - f(\vec y)\rVert_p$$

One can also speak of the "local" sensitivity of a function *at* at point
$$\vec{x}\in\mathbb{N}^{|\mathcal{X}|}$$.
This can be viewed as the global sensitivity where
$$\vec{x}\in\mathbb{N}^{|\mathcal{X}|}$$ is *fixed*.
An $$\ell_1$$ version of this is mentioned in definition 7.1.
As no notation is given, I will write $$\Delta_{p, \vec x}(f)$$.

> The local $$\ell_p$$ sensitivity of a function
> $$f : \mathbb{N}^{|\mathcal{X}|} \to \mathbb{R}^k$$ is:
> 
> $$\Delta_{p, \vec x}(f) = \max_{\vec y\in\mathbb{N}^{|\mathcal{X}|} : \lVert
> \vec x - \vec y\rVert_1 = 1} \lVert f(\vec x) - f(\vec y)\rVert_p$$

One can easily see that $$\Delta_p(f) = \max_{\vec x\in\mathbb{N}^{|\mathcal{X}|}}\Delta_{p, \vec x}(f)$$.
As the global sensitivity is more common, I'll quickly describe the motivation
for it.

Generally this is explained in a few steps.
First, it is stated (or already known) that $$\mathbb{N}^{|\mathcal{X}|}$$ is
the natural domain of an arbitrary database.
Intuitively, a *record* in a database is an element $$r\in\mathcal{X}$$, for some
finite set $$\mathcal{X}$$.
Then the *database* is a collection of records, where you count the multiplicity
of each record.
It therefore lives in $$\mathbb{N}^{|\mathcal{X}|}$$.
Next, one notes that the condition that $$\lVert \vec x-\vec y\rVert_1 = 1$$ can be
interpreted as "the databases $$\vec x$$ and $$\vec y$$ are neighboring", in that they
differ in how many times a particular record occurs by a count of 1.
Finally, the condition $$\lVert f(\vec x) - f(\vec y)\rVert_p$$ is interpreted as
quantifying how much $$f(\vec x) - f(\vec y)$$ can change on neighboring records.
The local sensitivity still bounds how much $$f$$ can change on neighboring
records, but it *in particular* bounds how much $$f$$ can change on records
which neighbor $$\vec x$$.

Given language like "The $$\ell_p$$ sensitivity quantifies how much a function
$$f$$ can change on adjacent points", it seems entirely unsurprising that you
can characterize it in terms of a derivative-like concept.
But what *is* a derivative for a function with the domain $$\mathbb{N}^{|\mathcal{X}|}$$?

## Discrete derivatives

Throughout this section, I will define "discrete" analogus of the:

1. Derivative $$f'$$
2. Partial Derivative $$(\partial/\partial x_i)f$$
3. Gradient $$\nabla f$$

I will do this by defining a (reasonable) "discrete" analogue of $$f'$$, and
then showing that in the continuous case, one can define $$(\partial/\partial
x_i)f$$ abstractly in terms of $$f'$$, and $$\nabla f$$ abstractly in terms of
$$(\partial/\partial x_i)f$$.
I'll then shove the discrete version of $$f'$$ through this set of
transformations to end up with a discrete version of $$\nabla f$$.
This "pipeline" is the sense in which I mean that my definition of the discrete
gradient is *natural*.

To highlight the similarities throughout this, I will use the same notation for
discrete and continuous derivatives.
If one wanted to actually compute with these, you could just check the domain of
$$f$$ to see which of the two methods to apply.
Anyway, onwards to technical content.

First, I'll restrict to "1 dimensional" functions $$f :
\mathbb{N}\to\mathbb{R}$$, before generalizing to something more useful.

Recall that the (continuous) derivative is traditionally defined via the
*difference quotient*:

$$f'(x) = \lim_{h\to 0, h\neq 0}\frac{f(x+h) - f(x)}{h}$$

If we want to restrict to discrete domains, we have to make sure that $$x+h$$ is
discrete (as it must be in the domain of $$f$$).
In our setting, this means that $$h\in\mathbb{N}$$.
As we need $$h\neq 0$$, this itself really means that we need $$h = 1$$, so can
define:

$$f'(x) = \frac{f(x+1) - f(x)}{1} = f(x+1) - f(x)$$

This definition is fairly standard (known as the [forward difference
operator](https://en.wikipedia.org/wiki/Finite_difference)), and denoted
$$\Delta(f)$$.
As $$\Delta$$ is *already* a super overloaded symbol in this post, I will
instead refer to it as $$f'(x)$$.

How do we generalize (continuous) derivatives to higher dimensions?
The way that most people learn is the [partial
derivative](https://en.wikipedia.org/wiki/Partial_derivative#Formal_definition):

> Let $$f : \mathbb{R}^n\to \mathbb{R}$$.
> The *partial derivative* of $$f$$ with respect to the variable $$x_i$$ is defined
as:
>
> $$\frac{\partial}{\partial x_i}f(\vec{x}) = \lim_{h\to 0, h\neq 0}
> \frac{f(\vec{x} +he_i) - f(\vec{x})}{h}$$

I will interpret the partial derivative as the 1D derivative of a particular
unary function.
The particular function is:

$$F_{\vec x, i}(t) = f(\vec x - \vec{x}_i + e_i + te_i)$$

One can check that in the continuous case this is such that $$F_{\vec x,
i}'(\vec x_i) = (\partial/\partial x_i)f(\vec x)$$.

When applied to the discrete derivative, one gets that:

> Let $$f : \mathbb{N}^n\to \mathbb{R}$$.
> For any $$\vec{x}\in\mathbb{N}^n$$ and $$i\in[n]$$, let $$F_{\vec{x}, i}(y) = f(\vec x - \vec{x}_ie_i + te_i)$$
> The *discrete partial derivative* of $$f$$ with respect to the variable
> $$x_i$$ is defined as:
>
> $$\begin{aligned}
 (\partial/\partial x_i)f(\vec{x}) &= F_{\vec{x}, i}'(\vec{x}_i)\\
&= f(\vec x + e_i) - f(\vec x)
 \end{aligned}$$

I'll quickly mention that (something close to this) [seems like it is used *some places*](https://en.wikipedia.org/wiki/Finite_difference#Multivariate_finite_differences).
For further motivation for why this
may be a sensible "discrete partial derivative", I'll point to a conceptually close definition of "$$\mathbb{F}_2$$-partial derivatives", which take a similar form (such as in [this
paper](http://web.cs.ucla.edu/~sherstov/pdf/directional.pdf), near theorem 1.5).

Of course, sensitivity is defined for functions $$f :
\mathbb{N}^{|\mathcal{X}|}\to\mathbb{R}^k$$, so we have one final step of
generalization to do.
Recall that the gradient is just a vector containing the partial derivatives in
each direction:

$$\nabla f(x) = ((\partial/\partial x_1)f(x),\dots, (\partial/\partial
x_n)f(x))$$

I will then define the "discrete gradiant" in the same way below:

$$\nabla f(x) = ((\partial/\partial x_1)f(x), \dots, (\partial/\partial x_n)f(x))$$

Before proceeding, we need to note that (when the codomain of $$f$$ is dimension
    $$>1$$) $$\nabla f(x)$$ is most naturally a *matrix* (called the [*Jacobian* matrix](https://en.wikipedia.org/wiki/Jacobian_matrix_and_determinant)), whose $$i$$th column
is $$(\partial/\partial x_i)f(x)$$.
I will still notate this $$\nabla f$$
I will define the norm of a matrix to be its [*operator
norm*](https://en.wikipedia.org/wiki/Operator_norm).
The general definition of this is included below:


> Let $$M : V\to W$$ be a linear operator, and let $$\lVert\cdot\rVert_V$$,
> $$\lVert\cdot\rVert_W$$ be norms on $$V$$ and $$W$$ respectively.
> Define the norm of $$M$$ by:
>
> $$\lVert M\rVert = \sup\{\lVert Mv\rVert_W \mid v\in V, \lVert v\rVert_V =
> 1\}$$

Of course, our domain is $$\mathbb{N}^{|\mathcal{X}|}$$.
What norm are we picking on it?
This choice is key to the entire construction, so I will discuss it in a little
more depth.

## Viewing $$\mathbb{N}^{|\mathcal{X}|}$$ as a normed space

The natural norm to put on $$\mathbb{N}^{|\mathcal{X}|}$$ is a norm induced by
viewing it as a subset of $$\mathbb{R}^{|\mathcal{X}|}$$.
This is precisely the choice I will make.
If this is so straightforward, why does this have its own separate section?

What is *less* straightforward is what the (closed) unit ball on
$$\mathbb{N}^{|\mathcal{X}|}$$ looks like.
Or actually, it's a little *too straightforward*.
This is clearly the intersection of the unit ball in $$\mathbb{R}^{|\mathcal{X}|}$$ with $$\mathbb{N}^{|\mathcal{X}|}$$.
What is a little odd is that, for any $$p\in (1,\infty)$$, it is *also* clear
that the only points in this ball are 0, along with the "standard basis vectors"
$$e_i$$.
Quickly notice that:

1. In the $$\ell_\infty$$ norm this isn't the case (for example, $$(1,1,\dots,
   1)$$ is in the ball).

2. This is an "easy" example of where the closed unit ball (things of norm
   $$\leq 1$$) is not the same as
   the closure of the open unit ball (things with norm $$<1$$).
   The open unit ball, in any $$\ell_p$$ norm, will just be $$\{0\}$$.

This is all to say that for $$p\in(1,\infty)$$, we have that:

$$\begin{aligned}
\{\vec x\in\mathbb{N}^{|\mathcal{X}|} \mid \lVert \vec x\rVert_p \leq 1\} &= \{0\} \cup \{\vec x\in\mathbb{N}^{|\mathcal{X}|} \mid \lVert \vec x\rVert_p = 1\}\\
&= \{0\}\cup \{e_i\mid i\in[|\mathcal{X}|]\}
\end{aligned}$$

The above implies that when we define the operator norm, the *particular* norm
we choose on the domain does not really matter (as long as it is not the
$$\ell_\infty$$ norm), as the elements of norm exactly 1 will always be
precisely the standard basis of the "overlying" vector space.
For this reason, we will write:

$$\begin{aligned}
\lVert M\rVert_p &= \sup\{\lVert Mv\rVert_p \mid
v\in\mathbb{N}^{|\mathcal{X}|}, \lVert v\rVert_p = 1\}\\
&= \sup\{\lVert Mv\rVert_p \mid v\in\{e_i\}, i\in[|\mathcal{X}|]\}
\end{aligned}$$

To denote the operator norm of a linear map $$M :
\mathbb{N}^{|\mathcal{X}|}\to\mathbb{R}^k$$.

We could be putting *any* $$\ell_p$$ norm on the domain while doing this, and
will end up with an equivalent definition.
We'll next show how this leads to a quite natural interpretation of the
$$\ell_p$$ sensitivity as the maximum of the operator norm of $$\nabla f(\vec{x})$$ over $$\vec{x}\in\mathbb{N}^{|\mathcal{X}|}$$.

# The $$\ell_p$$ sensitivity as a norm bound on the Jacobian matrix

Let $$f : \mathbb{N}^{|\mathcal{X}|} \to \mathbb{R}^k$$.
We will next show the following:

> $$\Delta_{\vec x, p}(f) = \lVert \nabla f(\vec x)\rVert_p$$

By taking maxes over $$\vec x$$, this also implies our expression for the global
sensitivity.
The proof is rather straightforward.
By the characteriation of norm-1 elements of $$\mathbb{N}^{|\mathcal{X}|}$$, we
have that the operator norm will be:

$$\lVert \nabla f(\vec x)\rVert_p = \sup\{\lVert [\nabla f(\vec x)] e_i\rVert_p\mid i\in[|\mathcal{X}|]\}$$

This is clearly equivalent to $$\max_{e_i}\lVert \nabla [f(\vec x)] e_i\rVert_p$$.
Note that $$[\nabla f(\vec x)] e_i$$ is the $$i$$th column of $$\nabla f(\vec x)$$, which is precisely $$(\partial/\partial x_i)f(\vec x)$$.
It follows that:

$$\begin{aligned}
\lVert \nabla f(\vec x)\rVert_p &= \max_{i}\lVert (\partial/\partial x_i)f(\vec x)\rVert_p
&= \max_i \lVert f(\vec x + e_i) - f(\vec
x)\rVert_p
\end{aligned}$$

This is *nearly* the $$\ell_p$$ sensitivity, except we are maximizing over the
choice of $$\vec x + e_i$$, rather than $$\vec y$$
such that $$\lVert \vec x - \vec y\rVert_1 = 1$$.
But these can readily be seen to be equivalent --- as norm 1 elements in
$$\mathbb{N}^{|\mathcal{X}|}$$ are precisely standard basis vectors, we have
that $$\vec x, \vec y$$ that are adjacent are of the form $$\vec x = \vec y \pm
e_i$$ for some $$i$$.

It follows that:

$$\Delta_{\vec x, p}(f) = \lVert \nabla f(\vec x)\rVert_p$$

and:

$$\Delta_{p}(f) = \max_{\vec x\in\mathbb{N}^{|\mathcal{X}|}} \Delta_{\vec x,
p}(f) = \max_{\vec x\in\mathbb{N}^{|\mathcal{X}|}} \lVert \nabla f(\vec
x)\rVert_p$$

# An explicit calculation in this framework

To flesh this out a little more, I'll next do a sensitivity calculation in the
above language.
I imagine this will look essentially like a sensitivity calculation in the
"standard" setting.

Recall that a database $$\vec{x}\in \mathbb{N}^{|\mathcal{X}|}$$ is a count of
the multiplicities of various records $$r\in\mathcal{X}$$.
What does a function of a databse look like then?
For example, if we have a database of names + ages, and want to compute the
average age, how do we do this in the "histogram" representation?

There are at least two ways to interpret this --- as I do essentially no differential privacy, I have no clue which is correct.
But the first is much nicer to work with, so I will use it (after briefly
discussing both options).

1. A database of ages can be identified with a vector $$\vec x = (x_0,x_1,\dots,
   x_{122})$$, where $$122$$ is an absolute upper bound on the age of a
   person (chosen as it is the [oldest verified
   age](https://en.wikipedia.org/wiki/List_of_the_verified_oldest_people)).

2. A database of peoples ages can be identified with a vector of *tagged ages*
   $$(x_{n_0, 0}, x_{n_0, 1},\dots, x_{n_0, 122}, x_{n_1,0},\dots)$$, where
   $$n_i$$ are the collection of *admissible names*.

While the second definition defines the database itself better (and is a *much*
more useful definition if we want to query the average number of vowels in each
persons name, or something else), the first leads to significantly "smaller"
databases generically.
Maybe viewing databases as multi-sets instead of vectors is what makes the
second the more appropriate definition --- I don't know.
All I know is I want to compute the sensitivity of the average age of someone,
and the first definition is easier to work with, so I will look at it for this
example.

The average age under the first definition is:

$$\mathsf{mean}(\vec{x}) = \frac{\sum_{i = 0}^{122} i \vec{x}_i}{\sum_{i = 0}^{122} \vec{x}_i} \sim \frac{123\mathbb{E}[\vec x]}{\lVert \vec x\rVert_1}$$

Here, the expectation is taken with respect to a uniform distribution on
$$\{0,1,\dots, 122\}$$ (a set of size $$123$$).
I use it mostly as compact notation for the sum.
This still has a fairly sensible discrete gradient:

$$\begin{aligned}
(\partial/\partial x_i)f(\vec x) &= \mathsf{mean}(\vec x + e_i) - \mathsf{mean}(\vec x) \\
&=\frac{123\mathbb{E}[\vec x + e_i]}{\lVert \vec x + e_i\rVert_1} -
\frac{123\mathbb{E}[\vec x]}{\lVert \vec x\rVert_1}\\
&= 123\left(\frac{\mathbb{E}[\vec x] + i}{\lVert \vec x\rVert_1 + 1} -
\frac{\mathbb{E}[\vec x]}{\lVert \vec x\rVert_1}\right)\\
&= 123\left(\frac{\lVert \vec x\rVert_1\mathbb{E}[\vec x] + i\lVert \vec x\rVert_1 -
(\lVert \vec x\rVert_1 + 1)\mathbb{E}[\vec x]}{\lVert \vec x\rVert_1(\lVert \vec
x\rVert_1 + 1)}\right)\\
&= 123\frac{i - \mathbb{E}[\vec x]}{\lVert \vec x\rVert_1 + 1}\\
&= 123\frac{\mathbb{E}[e_i - \vec x]}{\lVert x \rVert_1 + 1}
\end{aligned}$$

Note that we can convert from $$\lVert \vec x + e_i\rVert_1 \to \lVert \vec
x\rVert_1 + 1$$ as $$\vec x\in\mathbb{N}^{|\mathcal{X}|}$$ (so the absolute value
in the definition of the $$\ell_1$$ norm is immaterial), but we cannot
necessairly convert back from $$\lVert \vec x \rVert_1 + 1$$ to $$\lVert \vec x - e_i\rVert_1 = \lVert e_i - \vec x\rVert_1$$.
We can instead write:

$$\begin{aligned}(\partial/\partial x_i)\mathsf{mean}(\vec x) &= 123\frac{\mathbb{E}[e_i - \vec
x]}{\lVert \vec x\rVert_1 + 1}\\
&= \frac{\lVert e_i - \vec x\rVert_1}{\lVert \vec x\rVert_1 + 1}\mathsf{mean}(e_i - \vec
x)\\
&= -\frac{\lVert \vec x - e_i\rVert_1}{\lVert \vec x + e_i\rVert_1}\mathsf{mean}(\vec x - e_i)
\end{aligned}$$

If we then want to compute the noise sensitivity from this, we need to compute:

$$\begin{aligned}\max_{\vec x \in\mathbb{N}^{|\mathcal{X}|}}\max_i \lVert (\partial/\partial
x_i)f(\vec x)\rVert_p&= \max_{\vec x\in \mathbb{N}^{|\mathcal{X}|}}\max_i
\frac{\lVert 123\mathbb{E}[\vec x - e_i]\rVert_p}{\lVert x + e_i\rVert_1}
\end{aligned}
$$

The mean is over the set of ages $$\{0,1,\dots, 122\}$$.
This mean can be maximized when $$i = 122$$ and $$\vec{x} = 0$$ (there are some
small issues with the resulting count being negative which I will ignore), where
the mean is $$1$$, and thus the sensitivity is 123, as expected.
I get the impression this is essentially the same computation you would do in
the "standard" framework though.

# Potential extension

One application of this representation is to the development of a *sensitivity calculus*.
Here, "calculus" means a framework for calculating things.
One reason differential calculus has been so successful is that derivatives
give a (mechanically) simple way to do computations with complex functions.
In particular, as derivatives behave well with respect to:

1. Sums
2. Products
3. Quotients
4. Composition

One can usually compute the derivative of an arbitrary function in terms of the
above "rules", along with the knowledge of some "simple" derivatives.
By framing sensitivity in terms of derivatives, there is a hope that one could
derive such a set of rules for *discrete derivatives*.
This could "automate" the computation of $$\nabla f$$ in sensitivity analysis,
by giving a way to write $$\nabla f$$ as an expression in (known) "simple"
derivatives.

Of course, sensitivity analysis still requires computing the operator norm of
this expression.
The operator norm is sub-additive and sub-multiplicative, which one can leverage
to get an *upper bound* on the sensitivity in terms of the sensitivities of the
aforementioned "simple" derivatives.
This upper bound is potentially loose though, which would hurt the applicability
of this method.
If the upper bound is tolerable, this should be relatively easy to program.

Unfortunately, the above reasoning seems to be erronous.
I've written up some details [here]({% post_url 2020-10-16-sensitivity-calculus %}), but "short story" is that the chain rule one can derive involves terms of the form:

$$f(g(n) + g'(n))$$

Its unclear how to upper bound the $$\ell_p$$ norm of this, which would be
required to derive even (loose) upper bounds on the operator norm.
Note that for "continuous" calculus the chain rule has terms of the form:

$$f'(g(n))g'(n)$$

So for a sub-multiplicative norm one could hope to get some upper bounds.
Feel free to read the writeup for more details, but as it ends in "failure" it
is likely not very interesting.
