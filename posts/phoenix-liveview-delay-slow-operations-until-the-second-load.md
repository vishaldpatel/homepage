---
title: "How to Delay slow operations until the second render in Phoenix LiveView"
date: 2025-01-25T06:31:09Z
updated: 2025-01-26T21:52:36Z
slug: phoenix-liveview-delay-slow-operations-until-the-second-load
published: true
---

# How to Delay slow operations until the second render in Phoenix LiveView

## Update 2

I ran a little experiement to see which approach was quickest:

1. Load all queries fast and slow synchronously on `mount()` (slowest)
2. Load all queries synchronously but after the socket connects (fast!)
3. Loading all queries on `mount()` - the fast ones synchronously and the slow ones using `async_assign()` (faster)
4. Load everything after the socket connects, and use `async_assigns()` as well (fastest).

A combination approach (4) seems to be quickest becaues I saved by not duplicating quicker queries, and also loading slower queries asynchronously. Had my page been a public page, then I would have stuck with option 3 - Using async_assigns on mount.

Unless you're much more experienced than I am, using async_assigns can feel a bit tedious - so, reserving it for slower running queries seems to pay the most dividends. This is also why I like the combination approach.

## Update 1

After I published this post, I learned about [async assigns](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#module-async-assigns) from the amazing Elixir community on Reddit, which provide a more elegant solution to dealing with slow loading data. The following, then, is the less elegant approach.

## Original post

### TL;DR

```elixir
def mount(params, _session, socket) do
  # If this is the first pass
  if not connected?(socket) do
    # assigns for static render go here
    {:ok, render_with(&alt_render_method/1)}
  else
    # database loads and assigns for
    # dynamic render go here
    {:ok, socket}
  end
end
```

### Problem

You have a data heavy LiveView app that loads everything twice because LiveView does two passes:

1. Static render.
2. Establish Socket and Send Dynamic Render over socket.

This means that your LiveView's `mount()` method is called twice. Any big database queries are getting called twice. If your LiveView renders a data heavy dashboard, then this could slowdown your Dad's laptop from 2005 that you use for a webserver.

But, what if you don't need a full static render? Well, then you can delay the data loading until the second pass!

### Here's how

Since the socket isn't connected until the second pass, we can use `connected?/1` to check that. This let's us do something like:

```elixir
def mount(params, _session, socket) do
  if not is_connected?(socket) do
    {:ok,
     socket
     |> assign(
       basic_info_that_needs_to_be_shown: load_basic_stuff(),
       data_heavy_things: []
     )}
  else
    # dynamic load
    {:ok,
     socket
     |> assign(
       basic_info_that_needs_to_be_shown: load_basic_stuff(),
       data_heavy_things: load_all_the_heavier_stuff()
     )}
  end
end

defp load_basic_stuff(socket) do
  # ...
end

defp load_all_the_heavier_stuff(socket) do
  # ...
end
```

There's more that can be done here. For example, perhaps you'd want the static load to show a friendly loading screen. You can do something like the following:

```elixir
def mount(params, _session, socket) do
  if not is_connected?(socket) do
    # assigns for static render go here
    # instead of returning {:ok, socket}, use render_with() to render
    # different function for the static pass.
    {:ok,
     socket
     |> assign(
       basic_info_that_needs_to_be_shown: load_basic_stuff(),
       data_heavy_things: []
     )
     |> render_with(&alt_render/1)}
  else
    # dynamic load
    {:ok,
     socket
     |> assign(
       basic_info_that_needs_to_be_shown: load_basic_stuff(),
       data_heavy_things: load_all_the_heavier_stuff()
     )}
  end
end

def alt_render(assigns) do
  ~H"""
  <div>
    Loading....
  </div>
  """
end

def render(assigns) do
  # Your usual render
end
```

That's pretty much it.
