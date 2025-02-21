# Definition

A plug + helpers for overwriting the default `cache-control` header. The plug
supports all the response header directives defined in [RFC7234, section
5.2.2](https://datatracker.ietf.org/doc/html/rfc7234#section-5.2.2).

## Header directives

The `PlugCacheControl` plug takes a `directives` option which can specify either
_static_ or _dynamic_ header directives. Static directives are useful when you
don't need per-request directives. Static directives are defined very similarly
to a struct's key.

```elixir
plug PlugCacheControl, directives: [:public, max_age: {1, :hour}]
```

As seen in the above example, directive names with hyphens are mapped to atoms
by replacing the hyphens with underscores.

Boolean directives like `public`, `private`, `must-revalidate`, `no-store` and
so on can be included in the header value by simply including them in the
directives list e.g. no need for explicit `no_store: true` value. Note that as
per the standard, `no-cache` can also specify one or more fields. This is
supported via the definition below.

```elixir
plug PlugCacheControl, directives: [no_cache: ["somefield", "otherfield"]]
```

The `public` and `private` directives also have somewhat special handling so you
won't need to explicitly define `private: false` when you've used `:public` in
the "boolean section" of the directives list. Another important thing is that if
a directive is not included in the directives list, the directive will be
_omitted_ from the header's value.

The values of the directives which have a delta-seconds values can be defined
directly as an integer representing the delta-seconds.

```elixir
plug PlugCacheControl, directives: [:public, max_age: 3600]
```

A unit tuple can also be used to specify delta-seconds. The supported time units
are `second`, `seconds`, `minute`, `minutes`, `hour`, `hours`, `day`, `days`,
`week`, `weeks`, `year`, `years`. The following example shows how unit tuples
can be used as a conveniece to define delta-seconds.

```elixir
plug PlugCacheControl,
  directives: [
    :public,
    max_age: {1, :hour},
    stale_while_revalidate: {20, :minutes}
  ]
```

Dynamic directives are useful when you might want to derive cache control
directives per-request. Maybe there's some other header value which you care
about or a dynamic configuration governing caching behaviour, dynamic directives
are the way to go.

```elixir
plug PlugCacheControl, directives: &__MODULE__.dyn_cc/1

# ...somewhere in the module...

defp dyn_cc(\_conn) do
  [:public, max_age: Cache.get(:max_age)]
end
```

As seen in the previous example, the only difference between static and dynamic
directives definition is that the latter is a unary function which returns a
directives list. The exact same rules that apply to the static directives apply
to the function's return value.

## A note on behaviour

The first time the plug is called on a connection, the existing value of the
Cache-Control header is _replaced_ by the user-defined one. A private field
which signifies the header value is overwritten is put on the connection struct.
On subsequent calls of the plug, the provided directives' definitions are
_merged_ with the header values. This allows the user to build up the
Cache-Control header value.

Of course, if one wants to replace the header value on a connection that has an
already overwritten value, one can use the
`PlugCacheControl.Helpers.put_cache_control` function or provide a `replace:
true` option to the plug.

``` elixir
plug PlugCacheControl, directives: [...], replace: true
```

The latter approach allows for a finer-grained control and conditional
replacement of header values.

``` elixir
plug PlugCacheControl, [directives: [...], replace: true] when action == :index
plug PlugCacheControl, [directives: [...]] when action == :show
```

## Installation

Package is [available in Hex](https://hex.pm/docs/plug_cache_control) and can be
installed by adding `plug_cache_control` to your list of dependencies in
`mix.exs`:

```elixir
def deps do
  [
    {:plug_cache_control, "~> 1.0"}
  ]
end
```

Documentation can be generated with
[ExDoc](https://github.com/elixir-lang/ex_doc) and published on
[HexDocs](https://hexdocs.pm). Once published, the docs can be found at
[https://hexdocs.pm/plug_cache_control](https://hexdocs.pm/plug_cache_control).
