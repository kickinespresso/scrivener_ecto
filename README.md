# Scrivener.Ecto

[![Build Status](https://travis-ci.org/drewolson/scrivener_ecto.svg?branch=master)](https://travis-ci.org/drewolson/scrivener_ecto) [![Hex Version](http://img.shields.io/hexpm/v/scrivener_ecto.svg?style=flat)](https://hex.pm/packages/scrivener_ecto) [![Hex docs](http://img.shields.io/badge/hex.pm-docs-green.svg?style=flat)](https://hexdocs.pm/scrivener_ecto)

Scrivener.Ecto allows you to paginate your Ecto queries with Scrivener. It gives you useful information such as the total number of pages, the current page, and the current page's entries. It works nicely with Phoenix as well.

First, you'll want to `use` Scrivener in your application's Ecto Repo. This will add a `paginate` function to your Repo. This `paginate` function expects to be called with, at a minimum, an Ecto query. It will then paginate the query and execute it, returning a `Scrivener.Page`. Defaults for `page_size` can be configured when you `use` Scrivener. If no `page_size` is provided, Scrivener will use `10` by default.

You may also want to call `paginate` with a params map along with your query. If provided with a params map, Scrivener will use the values in the keys `"page"` and `"page_size"` before using any configured defaults.

Note: Scrivener.Ecto only supports Ecto backends that allow subqueries (e.g. PostgreSQL).

## Example

```elixir
defmodule MyApp.Repo do
  use Ecto.Repo, otp_app: :my_app
  use Scrivener, page_size: 10
end
```

```elixir
defmodule MyApp.Person do
  use Ecto.Schema

  schema "people" do
    field :name, :string
    field :age, :integer

    has_many :friends, MyApp.Person
  end
end
```

```elixir
def index(conn, params) do
  page =
    MyApp.Person
    |> where([p], p.age > 30)
    |> order_by(desc: :age)
    |> preload(:friends)
    |> MyApp.Repo.paginate(params)

  render conn, :index,
    people: page.entries,
    page_number: page.page_number,
    page_size: page.page_size,
    total_pages: page.total_pages,
    total_entries: page.total_entries
end
```

```elixir
page =
  MyApp.Person
  |> where([p], p.age > 30)
  |> order_by(desc: :age)
  |> preload(:friends)
  |> MyApp.Repo.paginate(page: 2, page_size: 5)
```

### More Examples

Simple paginate query based of the generated boiler plate `list_your_resource` function that the Phoenix Framework Generators create

```elixir
  alias  MyApp.Person
  ...
  
  @doc """
  Returns the list of people by page.

  ## Examples

      list_people_page(_params)
      [%Person{}, ...]

  """
  def list_people_page(params) do
    Repo.paginate(Person, params)
  end
  ```
  
Slightly more complex query that uses an Ecto query string  
  
```elixir
  
  alias  MyApp.Person
  ...
  
  @doc """
  Returns the list of people by page descending.

  ## Examples

      list_people_page_desc(_params)
      [%Person{}, ...]

  """
  def list_people_page_desc(params) do
    query = from(p in Person, order_by: [desc: :inserted_at])
    Repo.paginate(query, params)
  end
  
```  


## Installation

Add `scrivener_ecto` to your `mix.exs` `applications` and `dependencies`.

```elixir
def application do
  [applications: [:scrivener_ecto]]
end
```

```elixir
[{:scrivener_ecto, "~> 1.0"}]
```

## Contributing

First, you'll need to build the test database.

```elixir
MIX_ENV=test mix db.reset
```

This task assumes you have postgres installed and that your current user can create / drop databases. If you'd prefer to use a different user, you can specify it with the environment variable `SCRIVENER_ECTO_DB_USER`.

With the database built, you can now run the tests.

```elixir
mix test
```
