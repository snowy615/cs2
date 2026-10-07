# Models of Computation · Formula Sheet, Concepts Not to Confuse, Top Mistakes

*Quick reference for the whole course. [Overview](MoC-Summary.md) · [Part 1](MoC-1-Finite-Automata.md) · [Part 2](MoC-2-Regular-Expressions-Pumping-Myhill-Nerode.md) · [Part 3](MoC-3-Context-Free-Grammars.md) · [Part 4](MoC-4-Pushdown-Automata.md) · [Part 5](MoC-5-Turing-Machines.md) · [Part 6a](MoC-6a-Decidability.md) · [Part 6b](MoC-6b-Reductions-Rice-PCP.md) · [Part 7](MoC-7-Complexity.md)*

---

## 1. The machines

| Machine | Tuple | Transition function | Accepts $`w`$ iff |
|---|---|---|---|
| DFA | $`(Q,\Sigma,\delta,q_0,F)`$ | $`\delta: Q\times\Sigma\to Q`$ | $`q_0\xrightarrow{w}{}^*q\in F`$ |
| NFA | $`(Q,\Sigma,\delta,q_0,F)`$ | $`\delta: Q\times(\Sigma\cup\{\epsilon\})\to\mathcal{P}(Q)`$ | $`q_0\overset{w}{\Longrightarrow}q\in F`$ for *some* run |
| NPDA | $`(Q,\Sigma,\Gamma,\delta,q_0,\bot,F)`$ | $`\delta: Q\times(\Sigma\cup\{\epsilon\})\times\Gamma\to\mathcal{P}(Q\times\Gamma^*)`$ | $`(q_0,w,\bot)\overset{*}{\to}(q,\epsilon,\gamma)`$, $`q\in F`$ (final state) or $`\overset{*}{\to}(q,\epsilon,\epsilon)`$ (empty stack) |
| TM | $`(Q,\Sigma,\Gamma,\vdash,\sqcup,\delta,q_0,q_{\text{acc}},q_{\text{rej}})`$ | $`\delta: Q\times\Gamma\to Q\times\Gamma\times\{L,R\}`$ | $`(\epsilon,q_0,\vdash w)\overset{*}{\to}`$ a configuration with state $`q_{\text{acc}}`$ |
| $`k`$-tape TM | as TM | $`\delta: Q\times\Gamma^k\to Q\times\Gamma^k\times\{L,R\}^k`$ | same; single-tape simulation costs $`O(t^2)`$ |
| NDTM | as TM | $`\delta: Q\times\Gamma\to\mathcal{P}(Q\times\Gamma\times\{L,R\})`$ | some branch of the computation tree accepts; deterministic simulation $`2^{O(t)}`$ |
| CFG | $`(V,\Sigma,\mathcal{R},S)`$, rules $`A\to w\in V\times(V\cup\Sigma)^*`$ | $`uAv\Rightarrow uwv`$ | $`S\Rightarrow^*w`$, $`w\in\Sigma^*`$ |

**Step relations.** NPDA: $`(q,\gamma)\in\delta(p,a,A)`$ gives $`(p,av,A\beta)\to(q,v,\gamma\beta)`$ (pop $`A`$, push $`\gamma`$, consume $`a`$; $`a=\epsilon`$ consumes nothing). TM: $`\delta(q,b)=(q',c,L)`$ gives $`(ua,q,bv)\to(u,q',acv)`$; $`\delta(q,b)=(q',c,R)`$ gives $`(ua,q,bv)\to(uac,q',v)`$. TM conventions: $`\delta(p,\vdash)=(q,\vdash,R)`$; $`q_{\text{acc}},q_{\text{rej}}`$ are absorbing.

## 2. Language classes and their descriptions

| Class | Machines | Other descriptions | Example | Non-example |
|---|---|---|---|---|
| Regular | DFA = NFA | regex; right-linear CFG ($`A\to wB`$, $`A\to w`$); finite $`\equiv_L`$ index | ends in 1; $`0^n1^m`$; contains 10 | $`0^n1^n`$; equal 0s and 1s; $`ww`$; palindromes |
| Context-free | NPDA (non-deterministic!) | CFG; CNF ($`A\to BC`$, $`A\to a`$, $`S\to\epsilon`$) | $`0^n1^n`$; $`ww^R`$; balanced brackets; $`\overline{\{ww\}}`$ | $`0^n1^n0^n`$; $`a^nb^nc^n`$; $`ww`$; $`a^p`$ prime |
| Decidable (recursive) | total TM | RE ∩ co-RE | all of the above; $`A_{\text{DFA}},E_{\text{DFA}},EQ_{\text{DFA}},A_{\text{CFG}},E_{\text{CFG}}`$; PRIME | $`A_{\text{TM}}`$, HALT, $`E_{\text{TM}}`$, $`EQ_{\text{TM}}`$, $`EQ_{\text{CFG}}`$, PCP, TOTAL |
| r.e. (RE, semi-decidable) | TM | unrestricted grammar; λ-calculus; μ-recursive | $`A_{\text{TM}}`$, HALT, Hilbert's 10th, $`\overline{E_{\text{TM}}}`$ | $`\overline{A_{\text{TM}}}`$, $`\overline{\mathit{HALT}}`$, $`E_{\text{TM}}`$, TOTAL, $`\overline{\mathit{TOTAL}}`$ |

**Strict inclusions:** regular ⊂ CF ⊂ decidable ⊂ r.e. ⊂ all languages (witnesses: $`0^n1^n`$; $`0^n1^n0^n`$; $`A_{\text{TM}}`$; $`\overline{A_{\text{TM}}}`$). $`\mathcal{P}(\Sigma^*)`$ is uncountable, TMs are countable.

## 3. Closure properties

| Operation | Regular | CFL | Decidable | r.e. |
|---|---|---|---|---|
| union, concatenation, star | ✅ (ε-constructions / product) | ✅ ($`S\to S_1\mid S_2`$, $`S\to S_1S_2`$, $`S\to S_1S\mid\epsilon`$) | ✅ | ✅ |
| intersection | ✅ (product, $`F_1\times F_2`$) | ❌ ($`\{0^n1^n0^m\}\cap\{0^n1^m0^m\}`$) | ✅ | ✅ |
| complement | ✅ (swap $`F`$ in a **DFA**) | ❌ ($`\{ww\}`$ vs $`\overline{\{ww\}}`$) | ✅ (swap in a decider) | ❌ ($`A_{\text{TM}}`$) |
| ∩ with a regular language | ✅ | ✅ (NFA × NPDA product) | ✅ | ✅ |
| reversal, homomorphism | ✅ | ✅ | ✅ | ✅ |
| inverse homomorphism | ✅ | ✅ | ✅ | ✅ |
| $`\min,\max,\mathrm{init}`$ (prefix ops, tutorial 2) | ✅ | | | |

## 4. Constructions you must be able to write down

| Construction | Recipe |
|---|---|
| Product DFA ($`A_1\cup A_2`$) | $`Q_1\times Q_2`$; $`\delta((r_1,r_2),a)=(\delta_1(r_1,a),\delta_2(r_2,a))`$; start $`(q_1,q_2)`$; $`F=(F_1\times Q_2)\cup(Q_1\times F_2)`$ [∩: $`F_1\times F_2`$; $`\setminus`$: $`F_1\times(Q_2\setminus F_2)`$] |
| Subset construction (NFA → DFA) | states $`S\subseteq Q_N`$; start $`\{q : q_N\overset{\epsilon}{\Longrightarrow}q\}`$; $`S\xrightarrow{a}\{q' : \exists q\in S,\ q\overset{a}{\Longrightarrow}q'\}`$; accept iff $`S\cap F_N\neq\emptyset`$; build reachable subsets only |
| NFA union | new start $`q`$, $`\epsilon`$-edges to both starts; $`F=F_1\cup F_2`$ (tagged copies) |
| NFA concatenation | start of $`N_1`$; $`\epsilon`$-edges from each $`f\in F_1`$ to start of $`N_2`$; $`F=F_2`$ |
| NFA star | **new accepting start** $`q_0\xrightarrow{\epsilon}q_1`$; $`\epsilon`$-edges from each $`f\in F_1`$ back to the **old** start $`q_1`$; $`F=F_1\cup\{q_0\}`$ |
| Regex → NFA | base cases + the three constructions, by recursion on $`E`$ |
| NFA → regex (state elimination) | $`E^\emptyset_{q,q'}=a_1+\cdots+a_k`$ (or $`\emptyset`$), $`E^\emptyset_{q,q}=a_1+\cdots+a_k+\epsilon`$ (or $`\epsilon`$); $`E^X_{q,q'}=E^{X-r}_{q,q'}+E^{X-r}_{q,r}(E^{X-r}_{r,r})^*E^{X-r}_{r,q'}`$; $`L(M)=\sum_{f\in F}E^Q_{q_0,f}`$ |
| DFA → right-linear CFG | $`q\to aq'`$ iff $`\delta(q,a)=q'`$; $`q\to\epsilon`$ iff $`q\in F`$; start $`q_0`$ |
| Strongly right-linear CFG → NFA | $`R\xrightarrow{a}R'`$ iff $`R\to aR'`$; $`R\xrightarrow{\epsilon}R'`$ iff $`R\to R'`$; $`R\in F`$ iff $`R\to\epsilon`$ |
| Myhill–Nerode DFA | $`Q=\{[x]\}`$, $`q_0=[\epsilon]`$, $`\delta([x],a)=[xa]`$, $`F=\{[w] : w\in L\}`$ |
| CFG → NPDA | stack alphabet $`V\cup\Sigma\cup\{\bot\}`$; push $`S`$; pop variable ⇒ push a rule's RHS (leftmost on top, non-deterministic choice); pop terminal ⇒ match input; pop $`\bot`$ ⇒ accept |
| NPDA → one-state NPDA | stack symbols $`\langle pAq\rangle`$; start symbol $`\langle s\bot t\rangle`$; $`(q_0,B_1\cdots B_k)\in\delta(p,c,A)`$ ⇒ $`(*,\langle q_0B_1q_1\rangle\cdots\langle q_{k-1}B_kq_k\rangle)\in\delta'(*,c,\langle pAq_k\rangle)`$ for all $`q_1..q_k`$ |
| One-state NPDA → CFG | variables $`\Gamma`$, start $`\bot`$; rule $`A\to cB_1\cdots B_k`$ for each $`(q,B_1\cdots B_k)\in\delta(q,c,A)`$ |
| CFL ∩ regular | $`Q_1\times Q_2`$ NPDA with $`\delta((r,t),a,A)=\{((r',t'),\alpha) : r'\in\delta_1(r,a),(t',\alpha)\in\delta_2(t,a,A)\}`$ |
| $`k`$ tapes → 1 tape | $`\Gamma_N=\Sigma\cup\{\vdash\}\cup(\Gamma\cup\widehat\Gamma)^k`$: $`k`$ tracks, hatted symbol = head; each step = scan for hats, apply $`\delta`$, rewrite, return |
| NDTM → DTM | 3-tape BFS of the computation tree; $`O(t\,b^t)=2^{O(t)}`$ |
| Universal TM | tracks: $`\delta_M`$ / $`M`$'s tape / $`M`$'s state + head; simulate step by step |
| Decidable = RE ∩ co-RE | dovetail recognisers for $`L`$ and $`\overline L`$; accept if the first accepts, reject if the second accepts |

## 5. Theorems and their proof ideas

| Theorem | Proof idea |
|---|---|
| Every NFA has an equivalent DFA | subset construction; $`\subseteq`$: trace the DFA run, $`q_i\in S_i`$; $`\supseteq`$: work backwards from $`q_n\in S_n\cap F_N`$ |
| Kleene: regex ⇔ FA | ⇐ recursion on $`E`$ with the three constructions; ⇒ induction on $`\lvert X\rvert`$ with separating state $`r`$ |
| Kozen: axioms complete | soundness of rule 13 by induction on $`n`$ in $`fe_1\cdots e_n`$; completeness beyond scope |
| Pumping Lemma (regular) | $`p=\lvert Q\rvert`$; pigeonhole among $`q_0..q_p`$ gives a loop $`q_j\xrightarrow{y}{}^*q_j`$ with $`\lvert xy\rvert\le p`$, $`\lvert y\rvert>0`$ |
| Myhill–Nerode | (i) $`\hat\delta(q_0,x)=\hat\delta(q_0,y)\Rightarrow x\equiv_Ly`$ + pigeonhole; (ii) quotient DFA, $`\delta`$ well-defined since $`x\equiv_Ly\Rightarrow xa\equiv_Lya`$ |
| Regular ⇔ right-linear CFG | states ↔ variables |
| CFL ⇔ NPDA | A: leftmost derivation on the stack; B1: one-state NPDA with $`\langle pAq\rangle`$ (lemma: $`(*,x,\langle pAq\rangle)\overset{n}{\to}(*,\epsilon,\epsilon)\iff(p,x,A)\overset{n}{\to}(q,\epsilon,\epsilon)`$); B2: transitions → rules |
| Any CFL has a CNF grammar | (stated); derivations of length-$`n`$ words take $`2n-1`$ steps |
| CFL Pumping Lemma | tall parse tree repeats a variable; $`w=uxyzv`$, $`ux^iyz^iv\in L`$, $`\lvert xz\rvert>0`$, $`\lvert xyz\rvert\le p`$ |
| CFLs not closed under ∩ / complement | $`\{0^n1^n0^m\}\cap\{0^n1^m0^m\}=\{0^n1^n0^n\}`$; $`\overline{\{ww\}}`$ is CF via $`S\to AB\mid BA\mid A\mid B`$, $`A\to CAC\mid0`$, $`B\to CBC\mid1`$, $`C\to0\mid1`$ |
| TM = NDTM = multitape | simulations above |
| Not every language is r.e. | countable TMs vs uncountable $`\mathcal{P}(\Sigma^*)`$ (diagonal $`x_i\in L\iff x_i\notin L_i`$) |
| $`A_{\text{TM}}`$ r.e. | universal TM $`U`$ |
| $`A_{\text{TM}}`$ undecidable | $`D(\langle M\rangle)`$ = opposite of $`H(\langle M,\langle M\rangle\rangle)`$; $`D(\langle D\rangle)`$ contradicts |
| Decidable ⇔ RE ∧ co-RE | dovetailing; corollary $`\overline{A_{\text{TM}}}`$ not r.e. |
| HALT undecidable | decider $`H`$ for HALT ⇒ decider for $`A_{\text{TM}}`$ (if halts, simulate) |
| $`E_{\text{TM}}`$ undecidable | $`M_x`$ = "reject $`y\neq x`$; else run $`M`$ on $`x`$"; $`L(M_x)=\{x\}`$ or $`\emptyset`$; flip the answer (Turing reduction) |
| $`EQ_{\text{TM}}`$ undecidable | $`E_{\text{TM}}\le_m EQ_{\text{TM}}`$ via $`\langle M\rangle\mapsto\langle M,M_\emptyset\rangle`$ |
| TOTAL neither RE nor co-RE | $`\overline{\mathit{HALT}}\le_m\mathit{TOTAL}`$ ($`N_{M,x}`$: run $`M`$ on $`x`$ for $`\lvert y\rvert`$ steps; halted ⇒ loop, else accept) and $`\overline{\mathit{HALT}}\le_m\overline{\mathit{TOTAL}}`$ ($`N_{M,x}`$: erase $`y`$, run $`M`$ on $`x`$) |
| Mapping reduction theorems | $`A\le_m B`$: $`B`$ decidable ⇒ $`A`$ decidable; $`B`$ r.e. ⇒ $`A`$ r.e. (contrapositives give hardness) |
| Rice | $`N_{M,x}`$: save $`y`$; run $`M`$ on $`x`$; if halts run $`K`$ ($`\langle K\rangle\in P`$) on $`y`$; $`L(N_{M,x})\in\{L(K),\emptyset\}`$ ⇒ $`\mathit{HALT}\le_m P`$ |
| PCP undecidable | $`A_{\text{TM}}\le_m\mathit{MPCP}`$ (7 domino parts simulate $`M`$ on $`w`$, bottom one configuration ahead) and $`\mathit{MPCP}\le_m\mathit{PCP}`$ ($`\star`$-interleaving, $`[\star\diamond/\diamond]`$) |
| Linear speed-up | compress $`m`$ symbols into one; needs $`f(n)/n\to\infty`$ |
| $`k`$-tape $`t(n)`$ ⇒ 1-tape $`O(t^2)`$ | each simulated step scans tapes of length $`\le t(n)`$ |
| NDTM $`t(n)`$ ⇒ DTM $`2^{O(t)}`$ | $`\le b^{t(n)}`$ leaves, BFS |
| NP = poly-time verifiable | guess certificate / use certificate as the choice sequence |
| NP-complete $`B\in\mathbf{P}\iff\mathbf{P}=\mathbf{NP}`$ | every $`A\in\mathbf{NP}`$ has $`A\le_pB`$ |
| Cook–Levin: SAT NP-complete | formula encoding the $`n^k`$-step tableau of $`N_A`$ on $`w`$ |

## 6. Decidability status cheat-table

| Problem | DFA | CFG | TM |
|---|---|---|---|
| Acceptance $`A_{\text{X}}`$ | decidable (simulate) | decidable (CNF, $`2n-1`$ steps) | r.e., undecidable |
| Emptiness $`E_{\text{X}}`$ | decidable (mark reachable states) | decidable (mark generating variables) | co-r.e., undecidable |
| Equivalence $`EQ_{\text{X}}`$ | decidable (symmetric difference + emptiness) | undecidable | undecidable (neither r.e. nor co-r.e.) |
| Ambiguity | — | undecidable | — |
| Halting / totality | — | — | HALT r.e. undecidable; TOTAL neither |
| Non-trivial semantic property | — | — | undecidable (Rice) |

## 7. Complexity formulas

```math
f(n)=O(g(n))\iff\exists c,n_0\ \forall n\ge n_0:\ f(n)\le c\,g(n)\qquad
\mathbf{P}=\bigcup_{k\ge0}\mathrm{TIME}(n^k)\qquad
\mathbf{NP}=\bigcup_{k\ge0}\mathrm{NTIME}(n^k)
```

$`A\le_pB`$: poly-time computable $`f`$ with $`w\in A\iff f(w)\in B`$. NP-hard: $`\forall A\in\mathbf{NP}.\ A\le_pB`$. NP-complete: NP-hard and in NP. Verifier: $`A=\{w : V \text{ accepts } \langle w,c\rangle \text{ for some } c\}`$, $`V`$ polynomial in $`\lvert w\rvert`$. Examples: $`\{0^n1^n\}\in\mathrm{TIME}(n^2)`$ (1 tape), $`O(n)`$ (2 tapes); PATH, RELPRIME, PRIME (AKS, $`O(\ln^6n)`$), all CFLs ∈ P; HAMPATH, SAT ∈ NP (NP-complete).

## 8. Concepts you must not confuse

| This | vs that |
|---|---|
| **r.e. / recognise** (accept all of $`L`$, may loop outside) | **decidable / decide** (total: halt on everything) |
| **undecidable** (no decider) | **not r.e.** (no recogniser at all); $`A_{\text{TM}}`$ is undecidable *and* r.e. |
| **co-RE** (complement is r.e.) | "complement is decidable" (that would make it decidable) |
| Pumping Lemma (**necessary** for regular; proves non-regularity only) | Myhill–Nerode (**iff**; also gives minimal DFA size) |
| **you** choose $`s`$ and $`i`$ | **adversary** chooses $`p`$ and the split $`xyz`$ |
| DFA complement = swap $`F`$ | NFA/looping TM: swapping does **not** complement |
| $`\epsilon\in A^*`$ always | $`\epsilon\in A`$ only sometimes; $`E+\epsilon\not\equiv E`$ in general |
| $`01^*=0(1^*)`$ | $`(01)^*`$ |
| derivation (a sequence) | parse tree (an equivalence class of derivations); ambiguity = two **trees** / two **leftmost** derivations |
| ambiguous grammar | inherently ambiguous language |
| NPDA accept by final state (any stack) | accept by empty stack (any state); equivalent conventions |
| DPDA | NPDA (strictly stronger; CFL = NPDA) |
| NPDA transition "$`a,Z\to\gamma`$" **pops** $`Z`$ | "reads" $`Z`$ (write $`Z\to Z`$ to keep it) |
| TM **loops** | TM rejects |
| Turing reduction (use a decider as a subroutine, may flip) | mapping reduction (one computable $`f`$, $`w\in A\iff f(w)\in B`$) |
| reduce known-hard $`A`$ **to** new $`B`$ ($`A\le B`$) | the useless direction $`B\le A`$ |
| Rice: property of $`L(M)`$ | property of the machine $`M`$ (states, running time, halting, rejecting) |
| universal TM: **one** $`U`$ for all $`M`$ | NDTM simulation: a DTM **per** NDTM |
| **NP** = non-deterministic poly / poly-time verifiable | "not polynomial" |
| NP-hard (maybe outside NP) | NP-complete (NP-hard and in NP) |
| running time of NDTM = longest branch | size of the tree ($`2^{O(t)}`$, the simulation cost) |
| input size of PRIME = number of **digits** | the number $`n`$ itself (trial division is exponential) |

## 9. Top 20 mistakes

1. Drawing a "DFA" with a missing or duplicated transition (add a sink / it's an NFA).
2. Complementing an NFA by swapping accepting states.
3. Forgetting $`\epsilon`$ when computing a language ($`M_3`$ accepts $`\epsilon`$; $`A^*`$ contains $`\epsilon`$).
4. Star construction: reusing the old start as accepting (accepts `010` for $`\{0^i1\}^*`$).
5. Subset construction without $`\epsilon`$-closure, or building all $`2^n`$ subsets instead of the reachable ones.
6. Pumping Lemma: choosing $`s`$ not depending on $`p`$, choosing the split yourself, or ignoring $`\lvert xy\rvert\le p`$.
7. Concluding "regular" from the Pumping Lemma holding.
8. Myhill–Nerode with only finitely many distinguishable strings.
9. Precedence: $`01^*+1`$, $`0+10`$.
10. Proving ambiguity with two non-leftmost derivations of the same tree.
11. CNF with the start symbol on a right-hand side.
12. CFL Pumping Lemma pumping $`x`$ and $`z`$ with different exponents.
13. Assuming CFLs closed under ∩ or complement.
14. NPDA that forgets to pop/re-push, or pops $`\bot`$ too early.
15. Calling a looping TM a decider; swapping accept/reject in a non-total TM.
16. "Undecidable ⇒ not r.e."
17. Reduction in the wrong direction; or calling an answer-flipping Turing reduction a mapping reduction.
18. Applying Rice to machine properties (number of states, halting, rejects $`w`$) or trivial properties.
19. "NP = not polynomial"; "NP-hard ⊆ NP".
20. Measuring PRIME's input by the value rather than the number of digits.
