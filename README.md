## julia_port
example project to invoke julia functions in elixir to do scientific computing using port and metaprogramming

### prerequisite
* [julia](http://julialang.org/) installed and its access from shell
* in real_test: [`BackpropNeuralNet`](https://github.com/compressed/BackpropNeuralNet.jl) installed

### Installation

the package can be installed
by adding `julia_port` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [{:julia_port, "~> 0.1.0"}]
end
```

remark: if you install the package as a dependency, in order to run `script_test`,
you have to move ./deps/julia_port/julia and ./deps/julia_port/data to ./julia and ./data respectively
### usage
* simple_test: arithmetics 
```elixir
def simple_test(port) do
  port_send(port, "1+2")
  IO.puts port_receive(port, true)
end
```
* complex_test: linear algebra
```elixir
use GenFunction, [rand: 2, sum: 1, *: 2]

def complex_test(port) do
  rand(port, :a, 3, 3)
  rand(port, :b, 3, 3)
  JuliaPort.*(port, :c, :a, :b)
  port_receive(port, false)
  sum(port, :d, :c)
  IO.puts port_receive(port, true)
end
```
* real_test: neural network
```elixir
use GenFunction, [init_network: 1, train: 3, net_eval: 2]

def real_test(port) do
  port_send(port, "using BackpropNeuralNet")
  init_network(port, :net, [2, 3, 2])
  port_receive(port, false)
  train(port, :result1, :net, [0.15, 0.7], [0.1, 0.9])
  IO.puts port_receive(port, true)
  net_eval(port, :result2, :net, [0.15, 0.7])
  IO.puts port_receive(port, true)
end
```
* script_test: linear regression
```elixir
use GenFunction, [load_data: 1, lr_train: 2, lr_test: 3]

def script_test(port) do
  include_script(port, "./julia/lr.jl")
  load_data(port, {:x_train, :y_train}, "./data/train")
  load_data(port, {:x_test, :y_test}, "./data/test")
  lr_train(port, :beta, :x_train, :y_train)
  port_receive(port, false)
  lr_test(port, :error, :x_test, :y_test, :beta)
  IO.puts port_receive(port, true)
end
```
* run
```elixir
iex -S mix
port = JuliaPort.init
# => #Port<0.5310>
JuliaPort.simple_test port
# => received data: 3
JuliaPort.complex_test port
# => received data: 5.710327361153192
JuliaPort.real_test port
# => received data: 0.09117138901807831
# => received data: 2-element Array{Float64,1}: 0.381892 0.592638
JuliaPort.script_test port
# => received data: 0.9587912087912088
JuliaPort.terminate port
# => {#PID<0.143.0>, :close}
```
