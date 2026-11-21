# Calculations

Small scripts you can rerun (Elixir `.exs` run with `elixir`, using
`Mix.install/1` for any dependencies; no Python; plain shell is fine for
one-liners) for anything you'd otherwise work out on a napkin:
power budget, battery life, regulator dissipation, LED/pull-up resistor
values, trace width/current, signal timing.

Each script lists its inputs and assumptions at the top (with datasheet
references) and prints its results. When a part choice changes, rerun the
affected scripts.
