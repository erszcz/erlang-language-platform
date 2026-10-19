# Elixir support: rules for agents

## The support stays general

The Elixir support, like ELP, its Rust translator and the Scala checker
eqWAlizer, is a general tool for every Elixir project. Never add a rule,
AST rewrite or code transformation whose justification is that some
particular project then type-checks. A change is general when it is
correct for every project and follows from the semantics of Elixir or of
the checker, not from one codebase's idioms.

When a diagnostic on real code looks wrong or noisy:

- a checker imprecision is fixed in the checker, as a rule that holds for
  all input, never papered over where the input is produced;
- an Elixir standard-library spec or type that misleads the checker is
  overridden in this library, as below;
- a project's own code is the project's to fix.

## Standard-library overrides carry general Elixir knowledge

Elixir's standard library declares many types and specs in a way eqWAlizer
cannot use well. Overriding them is general, since it pertains to Elixir
as a whole, and it lives in this library and nowhere else:
`lib/eqwalizer_types.erl` for types, `lib/eqwalizer_specs.erl` for specs,
apart from the Erlang overrides in `src`, which Erlang projects use and
Elixir projects get from ELP's bundled copy.

## Rewrites of Elixir code or forms need explicit approval

ELP's service, `erlang_service_beam` in `erlang_service/src`, serves the
Erlang forms the Elixir compiler recorded. Any rewrite of Elixir code, or
of those forms, before they reach the Rust translator or the checker must
be put to the maintainer first: show the form before and after, and argue
from Elixir's semantics why the rewritten form is what the code means. A
rewrite that exists because the checker mishandles the original form is
not acceptable; fix the checker.

Two rewrites are approved so far:

- the bodies of compiler-generated functions are left out, keeping their
  specs and exports (`reject_generated_functions` in
  `erlang_service/src/erlang_service_beam.erl`);
- standard-library types Elixir declares as `any()` are overridden as
  `dynamic()`. Even there, prefer overriding the specs of the functions
  involved with type variables, and keep `dynamic()` only for what no
  spec can express.
