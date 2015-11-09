Lumberjack.jl
=============

[![Build Status](https://travis-ci.org/forio/Lumberjack.jl.png?branch=master)](https://travis-ci.org/forio/Lumberjack.jl)


## Quick Start
```julia
Pkg.add("Lumberjack")
```

### Create logs
```julia
julia> using Lumberjack

julia> debug("something innocuous happened!")
2013-12-02T19:39:16.123 UTC - debug:"something innocuous happened!"

julia> info("using more memory!", {:mem_allocated => 9001, :mem_left => 22})
2013-12-02T19:39:21.345 UTC - info:"using more memory!" mem_allocated:9001 mem_left:22

julia> warn("running really low on memory...", {:mem_left => "22 k"})
2013-12-02T19:39:44.456 UTC - warn:"running really low on memory..." mem_left:"22 k"

julia> try
         error("OUT OF MEMORY - IT'S ALL OVER - ARRGGGHHHH")
       catch err
         # Acts like Base.error, throws an `ErrorException`
       end
2013-12-02T19:39:48.678 UTC - error:"OUT OF MEMORY - IT'S ALL OVER - ARRGGGHHHH"

julia> log("info", "use `log` for user-defined modes, or to be verbose.")
2013-12-12T23:58:56.890 UTC - info:"use `log` for user-defined modes, or to be verbose."
```

### Add and remove `TimberTrucks`
Logs are brought to different output streams by `TimberTrucks`. To create a truck that will dump logs into a file, simply:
```julia
julia> Lumberjack.add_truck(LumberjackTruck("mylogfile.log"), "my-file-logger")
```
Now there is a truck named "my-file-logger", and it will write all of your logs to `mylogfile.log`. Your logs will still show up in the console, however, because -by default- there is a truck named "console" already hard at work. Remove it by calling:
```julia
julia> Lumberjack.remove_truck("console")
```

### Manage logging modes / levels
Each timber truck is configured to log messages _above_ a certain level / mode, and by default they will log everything. There are 4 built-in modes: `["debug", "info", "warn", "error"]`. To create a timber truck that will only record warnings and errors, you can:
```julia
julia> Lumberjack.add_truck(LumberjackTruck(STDOUT, "warn"), "dangerous-logger")
```

Or to configure an existing truck, you can call:
```
julia> configure(timber_truck; mode = "warn")
```

### Logging Options
`Lumberjack.add_truck` provides an optional third `Dict` argument. Possible keys are:

#### is_colorized
Colors can be added enabled using the following:

```julia
Lumberjack.add_truck(LumberjackTruck(STDOUT, nothing, {:is_colorized => true}), "console")
```

![colors](img/colors.png)

By default the following colors are used:
```julia
{"debug" => :cyan, "info" => :blue, "warn" => :yellow, "error" => :red}
```

#### colors
Custom colors/log levels can also be specified:
```julia
Lumberjack.add_truck(LumberjackTruck(STDOUT, nothing, :colors => {"debug" => :black, "info" => :blue, "warn" => :yellow, "error" => :red, "crazy" => :green}), "console")
```

#### uppercase
Log levels can be made uppercase (INFO vs info, etc.) with the following option:
```julia
Lumberjack.add_truck(LumberjackTruck(STDOUT, nothing, {:uppercase => true}), "console")
```

## Architecture

There are three main components of Lumberjack.jl that you can manipulate:

### LumberMill

A lumber mill holds information needed to manage the whole process of creating and storing logs. There is a global `_lumber_mill` inside the `Lumberjack` module and, in all likelyhood, you wont need to create another. All exported api methods that accept a `LumberMill` will be overloaded to use `_lumber_mill` by default.

### Saw functions

A saw function simply takes in a dict of parameters, adds or removes things, and then returns the dict back. By default, the `date_saw` is applied to logs that come in and appends `date => DateTime.now()`.

### TimberTruck

Timber trucks are used to send logs to their final destinations (files, the console, etc). A timber truck inherits from the abstract type `TimberTruck` and overloads the `log(t::TimberTruck, args::Dict)` function. By default, the framework will create a `LumberjackLog` truck that will print `args` as a string of `key:value` pairs to STDOUT.


## API

### Logging
```julia
log(lm::LumberMill, mode::String, msg::String, args::Dict)
```
+ `mode` is a string like "debug", "info", "warn", "error", etc
+ `msg` is an explanative message about what happened
+ `args` is an optional dictionary of data to be recorded alongside `msg`


```julia
debug(lm::LumberMill, msg::String, args::Dict)
info(lm::LumberMill, msg::String, args::Dict)
warn(lm::LumberMill, msg::String, args::Dict)
error(lm::LumberMill, msg::String, args::Dict)
```
+ each call `log` with `mode` filled in appropriately


### Saws
```julia
add_saw(lm::LumberMill, saw_fn::Function, index)
```
+ `index` is optional and will default to the end of the saw list


```julia
remove_saw(lm::LumberMill, index)
```
+ `index` is optional, by default the last saw in the list will be removed


```julia
remove_saws(lm::LumberMill)  # removes ALL saws currently in use
```


### Trucks
```julia
add_truck(lm::LumberMill, truck::TimberTruck, name)
```
+ `name` is optional and will default to a unique id


```julia
remove_truck(lm::LumberMill, name)
```
+ `name` is the id associated with the truck to be removed


```julia
remove_trucks(lm::LumberMill)  # removes ALL trucks currently in use
```


### Configuration
```julia
configure(lm::LumberMill; modes = ["debug", "info", "warn", "error"])
```
+ `modes` is an ordered array of logging levels


