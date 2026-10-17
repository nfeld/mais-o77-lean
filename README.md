# MAIS-O77: learning coefficients on matrix-factorization fibers and saddles

A Lean 4 development and accompanying mathematical manuscript for the
**two-sided band-volume formulation** of
[MAIS-O77](https://github.com/Math-for-AI-Safety/MAIS/blob/main/open-problems/MAIS-O77.md).

**Status — statement review reported 17 October 2026:** the published Lean
file has passed compilation and a separate `leanchecker` kernel replay. The
final theorem has no Wishart hypothesis. Niels Feld has personally reviewed
the main Lean statement and the upstream definitions needed to interpret it.
In his judgment, they faithfully express an answer to the natural-language
MAIS-O77 question in its two-sided band-volume formulation, under the stated
hypotheses.

This records Feld's specification review. Independent refereeing and complete
personal understanding of the proof remain separate tasks. The repository
remains open to scrutiny, reproduction, and correction. This documentation
update leaves the checked Lean file unchanged.

## Start here

| File | Purpose |
|---|---|
| [Manuscript PDF](mais-o77-blueprint_2026_10_17_15_54_UTC.pdf) | Part I is the human-facing preprint and specification guide; Part II gives detailed explanations alongside the Lean development. |
| [Manuscript TeX](mais-o77-blueprint_2026_10_17_15_54_UTC.tex) | Source of the manuscript. |
| [Lecture slides PDF](mais_o77_lectures_2026_10_17_15_54_UTC.pdf) | Five lectures for geometers: motivation, the formal statement and definitions, and a proof outline. |
| [Lecture slides TeX](mais_o77_lectures_2026_10_17_15_54_UTC.tex) | Editable Beamer source; quoted Lean declarations and a statement-audit index. |
| [Lean source](O77Full_2026_10_06_20_50_UTC.lean) | Single project file, requiring the pinned Mathlib environment below. |
| [Local checking package](o77-local_2026_10_06_22_09_UTC.tar.gz) | Podman scripts, setup instructions, statement-shape probes, and historical logs. |
| [7 October 2026 recheck log](recheck-20261007T222112Z-GHTD7O.log) | Source hashes, resolved dependencies, and successful build/replay exit statuses. |

## Mathematical scope

Let $I,O,H$ be natural numbers, with $I,O\geq 1$. For a target matrix
$\Phi\in\mathbb{R}^{O\times I}$ of rank $r<H$, consider
$A\in\mathbb{R}^{H\times I}$, $B\in\mathbb{R}^{O\times H}$, and

$$
L_\Phi(A,B)=\tfrac12\lVert BA-\Phi\rVert_F^2,
\qquad F_\Phi=\{(A,B):BA=\Phi\}.
$$

The parameter space carries product Lebesgue measure in the **original matrix
entries**, with the Euclidean norm obtained by summing the squared entries of
both factors. At a point $w=(A,B)$, the local pair $(\lambda,m)$ describes

$$
\mathrm{vol}\{z:\lVert z-w\rVert<\delta,\quad
 |L_\Phi(z)-L_\Phi(w)|<\varepsilon\}
\asymp \varepsilon^\lambda\bigl(\log(1/\varepsilon)\bigr)^{m-1}.
$$

Here $\lambda\geq0$ and $m\in\mathbb{N}$, $m\geq1$. The assertion is
**radius-first**: there exists $\delta_0>0$ such that for every
$0<\delta<\delta_0$, there are positive lower/upper comparison constants and
an $\varepsilon$ cutoff below $e^{-1}$ for which the two bounds hold for every
sufficiently small positive $\varepsilon$. These constants may depend on $w$
and $\delta$, but not on $\varepsilon$. Balls and band inequalities are strict.
The formalization uses this measured predicate, not a definition that simply
assigns the proposed formula to the phrase “learning coefficient.”

At an exact fit, write $a=\mathrm{rank}A$, $b=\mathrm{rank}B$, and set

$$
p=O-b,\quad q=I-a,\quad h=H+r-a-b,\quad s=Oa+b(I-a),
\qquad c(t)=(h-t)(q-t)+pt.
$$

Let $d_0$ be the minimum of $c(t)$ over integers $0\leq t\leq\min(h,q)$,
and let $\mu_0$ be the number of minimizing integers. The exact-fit pair is

$$
\left(\frac{s+d_0}{2},\mu_0\right).
$$

The result also characterizes the feasible rank pairs over each fixed target,
the entire minimum-threshold locus, the attained minimum value, and the
greatest multiplicity among threshold minimizers. Multiplicity need not be
constant on that locus.

For saddle rungs, the supplied singular vectors form orthonormal families with
strictly decreasing positive singular values. A rung consists of points whose
product is the corresponding singular truncation **and** whose two factors
annihilate the discarded directions. Every nonoptimal rung has pair $(1,1)
at every point. The theorem also gives rung inhabitation. When $r=0$, the
nonoptimal-rung assertion is vacuous.

Meromorphic continuation and the identification of this volume invariant with
a zeta pole are **not** formalized here. Neither are stochastic-gradient
dynamics or arbitrary-depth networks.

## The formal endpoint

The final theorem has type:

```lean
O77.answersO77 :
  ∀ {I O H : ℕ} (Phi : Matrix (Fin O) (Fin I) ℝ),
    0 < I → 0 < O → Phi.rank < H → O77.AnswersO77 (H := H) Phi
```

The six conjuncts of
[`O77.AnswersO77`](O77Full_2026_10_06_20_50_UTC.lean#L19954)
state the exact-fit pair, fixed-target rank realization, the minimum locus,
the attained minimum threshold, the greatest attained multiplicity on that
locus, and rung inhabitation with the all-points saddle pair.

The earlier conditional theorem `O77.answersO77_of_wishart` remains in the
file as an intermediate result. The final
[`O77.answersO77`](O77Full_2026_10_06_20_50_UTC.lean#L27307)
uses the proved
[`O77.GlobalSpectralIntegral.realWishartEigenvalueFormula`](O77Full_2026_10_06_20_50_UTC.lean#L27299).
Thus no unproved Wishart, chart, or local-pair premise is supplied to the final
theorem. Its dimension and rank hypotheses remain explicit.

## Reproduce the check

On Linux, with Git and rootless Podman available:

```bash
git clone https://github.com/nfeld/mais-o77-lean.git
cd mais-o77-lean
tar -xzf o77-local_2026_10_06_22_09_UTC.tar.gz
cd o77-local
chmod +x o77.sh
./o77.sh build
./o77.sh setup
./o77.sh doctor
./o77.sh recheck ../O77Full_2026_10_06_20_50_UTC.lean
```

The extracted package's README documents resource requirements and commands.
Setup downloads the dependencies; subsequent checking containers have network
access disabled and mount the submitted source read-only. The source is copied
and compared before building. Its host filename is unchanged; `O77Target` is
only its internal Lake module name.

| Dependency | Pin |
|---|---|
| Lean | `leanprover/lean4:v4.34.0-rc2` |
| Lean commit | `6a10ac8c22beadecabdbb0919c2b50214762f91d` |
| Mathlib commit | `664cbbe52927ce6acaa39ca35498336116e123b8` |

The SHA-256 of the published Lean file is:

```text
54dd6627468f7f1ad19784b8085e567806d0a087095a1f56a99bb920436acacc
```

The October 7 log records this hash both on the host and after copying, confirms
the pinned environment, and records zero exit statuses for the build, kernel
replay, and log writer. It contains no error or warning diagnostics. The Lake
build reused cached results; a separate module kernel replay then succeeded.

The source contains seven `#guard_msgs` axiom checks. The final theorem's guard
requires exactly `[propext, Classical.choice, Quot.sound]`; `sorryAx` and
additional project axioms are not permitted by that guard. Axiom reports must
be read together with theorem types: ordinary hypotheses are not listed as
axioms.

This run used default `leanchecker`, which replays the submitted module starting
from its imported environment. It uses Lean's own kernel, not an independently
implemented kernel. To replay imported constants too, run:

```bash
FRESH=1 ./o77.sh recheck ../O77Full_2026_10_06_20_50_UTC.lean
```

That is an additional check; the published October 7 log is not a `--fresh`
receipt. Selected statement-shape regression examples can also be run with
`./o77.sh probe FILE.lean`. They do not replace review of the underlying
definitions.

## Human review and feedback

The development was produced predominantly with language models under Niels
Feld's direction. On 17 October 2026 he reported that he had personally
reviewed `O77.answersO77`, its conclusion `O77.AnswersO77`, and the upstream
definitions needed to interpret them. In his judgment, these express the
intended two-sided band-volume O77 question under its stated hypotheses.

The reviewed source is `O77Full_2026_10_06_20_50_UTC.lean`, with the SHA-256
given above, in the previously announced
[repository snapshot](https://github.com/nfeld/mais-o77-lean/tree/f4766ac0a2875cce22aa835c59b6b02beb02b698).
The review concerns the original-entry loss, measure and norm; the
radius/epsilon quantifiers; rank arithmetic and the minimum locus; attained
extrema; and both annihilation clauses in saddle-rung membership.

Feld has also worked through the mathematical exposition, but does not yet
claim complete personal understanding or an independent verification of every
part of the proof. His specification review and the existing mechanical checks
provide distinct evidence. Independent review remains welcome. No Comparator
run is claimed.

Part I of the manuscript collects the formal statements and definitions. The
lecture slides introduce them more gradually and give a mathematical proof
outline; they do not replace the Lean source or its checking log. Reproduction
reports, mathematical corrections, and clearer exposition are welcome through
this repository's issues. Please identify the source filename or commit, and
include a complete log for checking failures.

## Development and acknowledgements

The manuscript credits **GPT Sol 5.6 and GPT Astra 6.0, directed by Niels Feld**.
The local checking harness was developed mainly with Claude Opus 5.0 and later
revised with OpenAI assistance.

The project builds on contributions already shared in the MAIS discussions:

- **Lionel Levine and the MAIS project**, for the problem and research agenda,
  with the AI contributions recorded in the original sources.
- **Rob Sneiderman (`Robby955`)**, for the overlapping exact-fit rank formula,
  matrix normal form, and residual calculation in
  [MAIS-O70, issue #3](https://github.com/Math-for-AI-Safety/MAIS/issues/3),
  developed with Claude Opus 5.
- **Svyatoslav Novikov (`kumino`)**, for the O77 candidate solution, including
  the all-points saddle result, in
  [issue #12](https://github.com/Math-for-AI-Safety/MAIS/issues/12),
  developed with OpenAI Codex and explicitly crediting Sneiderman's overlapping
  fiber result.
- **Mario Brcic (`mbrcic`)**, for the separate
  [AI Safety Formalization Atlas](https://github.com/mbrcic/ai-safety-formalization-atlas)
  and formalization/specification discussions in the MAIS issues.

These acknowledgements do not imply that those contributors have reviewed or
endorsed this repository. No priority claim is made here for the previously
posted fiber or saddle formulas. The manuscript contains the detailed literature
references and attribution.

## License

The original Lean code, scripts, configuration files, and code listings are
licensed under [Apache License 2.0](LICENSE). The manuscript and other original
non-code documentation are licensed under
[Creative Commons Attribution 4.0 International](LICENSE-DOCS).

Both licenses permit copying, modification, redistribution, and commercial use,
subject to their terms. See [LICENSING.md](LICENSING.md) for the scope and
attribution information. Third-party materials retain their own licenses.
