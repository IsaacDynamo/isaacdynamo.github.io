---
layout: post
title:  "Mockingbirds and Knuckledragger"
date:   2026-01-25 23:42:29 +0100
categories: math
---

Recently I discovered [Knuckledragger](https://www.philipzucker.com/knuckledragger/), a proof assistant in Python around Z3. I decided to try it to prove the first bird puzzle from [To Mock a Mockingbird by Raymond Smullyan](https://en.wikipedia.org/wiki/To_Mock_a_Mockingbird). I already had a proof for this in Lean, so I used it as a starting point. Knuckledragger recently got support for [Lean syntax](https://www.philipzucker.com/kd_microlean/), so that made porting a bit easier.

After some trial and error I ended up with this.

{% highlight py %}
import kdrag as kd
import kdrag.smt as smt

D = smt.DeclareSort("D")
app = smt.Function("app", D, D, D)

kd.notation.call.register(D, app)

mocking = kd.lean(r"∀ M : D, ∀ y : D, app M y = app y y")
fond = kd.lean(r"∀ (A B : D), app A B = B")
C1 = kd.lean(r"∀ (A B : D), ∃ C : D, ∀ x : D, app A (app B x) = app C x")
M = smt.Const("M", D)
C2 = smt.Exists([M], mocking(M))

@kd.Theorem(r"∀ A : D, ∃ B : D, app A B = B")
def fondness(pf: kd.tactics.ProofState):
    [A] = pf.intros()
    [M], hM = kd.tactics.skolem(kd.axiom(C2))
    [C], hC = kd.tactics.skolem(kd.axiom(C1)(A, M))
    pf.exists(M(C))
    pf.rw(hC(C))
    pf.rw(hM(C))
{% endhighlight %}

I was not quite satisfied with the result because I had to make `C1` and `C2` axioms. After reading more of the docs and various examples found on [Philip Zuckers blog](https://www.philipzucker.com/), I gave it a second try with `Lemma`. The `Lemma` object can take `C1` and `C2` as assumptions.

{% highlight py %}
l = kd.Lemma(kd.lean(r"∀ A : D, ∃ B : D, app A B = B"), assumes=[C1, C2])
A = l.fix()
M = l.obtain(1)
l.specialize(0, A, M)
C = l.obtain(2)
l.exists(M(C))
fondness2 = l.qed()
{% endhighlight %}

I think this is the intended way to construct the proof. However I prefer the indentation of the `Theorem` decorator. I looked through the codebase but there isn't a `Lemma` decorator. However the `Theorem` decorator seems to be a wrapper around `Lemma`. So it was easy to add an optional `assumes` argument to the `Theorem` decorator.

One other inconvenience is that the simple Lean parser doesn't let us call functions-like objects. It lets you known with a nice error. However support for this was also strait forward to hack into to the library, but I'm not sure how robust this is.

Now we can use the following code to construct the prove. Note that the lean parser now knowns that `mocking` is a function. And it knows about the call() overload on the `D` type, so `app A B` can be also be written as `A B`.

{% highlight py %}
C2 = kd.lean(r"∃ M, mocking M")

@kd.Theorem(r"∀ A : D, ∃ B : D, A B = B", assumes=[C1, C2])
def fondness3(l: kd.tactics.ProofState):
    A = l.fix()
    M = l.obtain(1)
    l.specialize(0, A, M)
    C = l.obtain(2)
    l.exists(M(C))
{% endhighlight %}

I'm happy with this result. I think it's a pretty compact and readable. I don't think that I took full advantage of Z3's power, but I still had a lot of fun playing with this theorem prover.

