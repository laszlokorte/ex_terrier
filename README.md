# Experiment with implementing Geometric Algebra in Elixir

This Livebook contains some experiments with Geometric Algebra.

For example the rendering of the 3D scene below as SVG via 3D Projective Geometric Algebra.

The implementation of the algebra itself can be found in the [Galixir](https://github.com/laszlokorte/galixir) [Hex package](https://hex.pm/packages/galixir).

![SVG Renderderd inside Livebook](./preview.png)

```ex
defmodule CameraUtils do
  import PGA3

  def align(ps, qs) do
    # https://observablehq.com/@enkimute/glu-lookat-in-3d-pga
    initial_m = one = PGA3.new(scalar: 1)
    initial_q = PGA3.dual(PGA3.new(scalar: 1))

    Enum.zip_reduce(ps, qs, {initial_m, initial_q}, fn p, q, {m, prev_q} ->
      p = prev_q |> PGA3.join(PGA3.transform(m, p)) |> normalize()
      new_q = prev_q |> PGA3.join(q) |> normalize() |> PGA3.blade_inverse()
      new_m = new_q |> PGA3.gp(p) |> add(one) |> PGA3.gp(m)

      {new_m, new_q}
    end)
    |> elem(0)
  end

  def look_at(
        position \\ point(0, 10, 0),
        target \\ point(0, 0, 0),
        pole \\ ideal_point(0, 0, 1)
      ) do
    align(
      [position, target, pole],
      [point(0, 0, 0), point(0, 0, 1), ideal_point(0, 1, 0)]
    )
  end
end
```
