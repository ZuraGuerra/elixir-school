---
layout: page
title: Concurrencia
category: advanced
order: 3
lang: es
---

Uno de los puntos más atractivos de Elixir es el manejo de concurrencia, ya que gracias a la ErlangVM es más sencilla de lo que podríamos pensar. El modelo de concurrencia depende de actores, que son procesos contenidos que se comunican entre ellos a través de mensajes.

En esta lección veremos los módulos de concurrencia que vienen en Elixir. En el siguiente capítulo cubriremos los comportamientos de OTP que los implementan.

## Índice

- [Procesos](#procesos)
  - [Envío de mensajes](#envío-de-mensajes)
  - [Procesos ligados](#procesos-ligados)
  - [Monitoreo de procesos](#monitoreo-de-procesos)
- [Agentes](#agentes)
- [Tareas](#tareas)

## Procesos

Los procesos de la ErlangVM son ligeros y se ejecutan en todos los procesadores. Aunque podrían parecer hilos de ejecución nativos, son más sencillos y en una aplicación de Elixir es común tener miles de estos de manera concurrente.

La manera más sencilla de crear un nuevo proceso es con `spawn`, que puede recibir una función anónima o nombrada; esto nos retorna un _identificador de proceso_ (en adelante, PID, por _Process Identifier_) para distinguirlo dentro de nuestra aplicación.

Para empezar, vamos a crear un módulo y a definir la función que queremos ejecutar:

```elixir
defmodule Example do
  def add(a, b) do
    IO.puts(a + b)
  end
end

iex> Example.add(2, 3)
5
:ok
```

Para evaluar la función asincrónicamente usamos `spawn/3`:

```elixir
iex> spawn(Example, :add, [2, 3])
5
#PID<0.80.0>
```

### Envío de mensajes

Los procesos se comunican enviándose mensajes y los componentes principales de esto son `send/2` y `receive`. La función `send/2` nos permite enviar mensajes a PIDs. Para escucharlos usamos `receive`, que también nos permite separar por casos según el contenido del mensaje; si no hay coincidencias, la ejecución continúa sin interrupciones.

```elixir
defmodule Example do
  def listen do
    receive do
      {:ok, "hello"} -> IO.puts "World"
    end
  end
end

iex> pid = spawn(Example, :listen, [])
#PID<0.108.0>

iex> send pid, {:ok, "hello"}
World
{:ok, "hello"}

iex> send pid, :ok
:ok
```

### Procesos ligados

Uno de los problemas de `spawn` es no saber cuando un proceso falla, por ello necesitamos ligar nuestros procesos usando `spawn_link`. Dos procesos ligados reciben notificaciones de salida entre ellos:

```elixir
defmodule Example do
  def explode, do: exit(:kaboom)
end

iex> spawn(Example, :explode, [])
#PID<0.66.0>

iex> spawn_link(Example, :explode, [])
** (EXIT from #PID<0.57.0>) :kaboom
```

A veces, no queremos que uno de los procesos ligados haga fallar al otro, por lo que necesitamos cachar las salidas; estas son recibidas como tuplas: `{:EXIT, from_pid, reason}`.

```elixir
defmodule Example do
  def explode, do: exit(:kaboom)
  def run do
    Process.flag(:trap_exit, true)
    spawn_link(Example, :explode, [])

    receive do
      {:EXIT, from_pid, reason} -> IO.puts "Exit reason: #{reason}"
    end
  end
end

iex> Example.run
Exit reason: kaboom
:ok
```

### Monitoreo de procesos

¿Qué tal si no queremos ligar dos procesos, pero de todas maneras mantenernos informados? Para eso podemos usar el monitoreo de procesos con `spawn_monitor`. Cuando monitoreamos un proceso, recibimos un mensaje si este falla sin que nuestro proceso actual lo haga también y sin tener que cachar explícitamente la salida.

```elixir
defmodule Example do
  def explode, do: exit(:kaboom)
  def run do
    {pid, ref} = spawn_monitor(Example, :explode, [])

    receive do
      {:DOWN, ref, :process, from_pid, reason} -> IO.puts "Exit reason: #{reason}"
    end
  end
end

iex> Example.run
Exit reason: kaboom
:ok
```

## Agentes

Los agentes son una abstracción de los procesos en segundo plano manteniendo su estado. Podemos accesarlos desde otros procesos dentro de nuestra aplicación y nodo. El estado de nuestro agente es igual al del valor de retorno de nuestra función:

```elixir
iex> {:ok, agent} = Agent.start_link(fn -> [1, 2, 3] end)
{:ok, #PID<0.65.0>}

iex> Agent.update(agent, fn (state) -> state ++ [4, 5] end)
:ok

iex> Agent.get(agent, &(&1))
[1, 2, 3, 4, 5]
```

Si nombramos a un agente, podemos hacer referencia a este por su nombre en vez de por su PID:

```elixir
iex> Agent.start_link(fn -> [1, 2, 3] end, name: Numbers)
{:ok, #PID<0.74.0>}

iex> Agent.get(Numbers, &(&1))
[1, 2, 3]
```

## Tareas

Las tareas nos permiten ejecutar una función en segundo plano y recuperar su valor de retorno después. Pueden sernos particularmente útiles para manejar operaciones costosas sin bloquear la ejecución de la aplicación.

```elixir
defmodule Example do
  def double(x) do
    :timer.sleep(2000)
    x * 2
  end
end

iex> task = Task.async(Example, :double, [2000])
%Task{pid: #PID<0.111.0>, ref: #Reference<0.0.8.200>}

# Haz algo más mientras

iex> Task.await(task)
4000
```
