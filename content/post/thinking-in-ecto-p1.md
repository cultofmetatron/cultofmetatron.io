---
Categories:
- Development
- Elixir
- Phoenix
Description: ""
Tags:
- Development
- elixir
date: 2017-04-22T17:02:46-04:00
menu: blog
title: Thinking in Ecto - schemas and changesets
---

> Note: I'm basing much of this code paths on the new phoenix 1.3 RC1.

I started writing elixir 6 months ago and it's been pure love.
Ecto specifically, makes developing saas applications a real joy.

[Ecto](https://github.com/elixir-ecto/ecto) is the database abstraction layer for elixir and comes baked into phoenix.
Unlike an ORM like activerecord or rails, ecto provides a set of macros that expose a dsl layer for operating on your database similar to .net's linq. Instead of models acting as a proxy interface to your database translating imperative code into declarative sql searches, Ecto exposes the notion of Schemas, Changesets and Queryables.

## Schema

Lets say we want a set of users.
You start by creating a schema which tells ecto how to interact with the database concerning a certain type. You start with a file user.ex. By convention, you should place it in `lib/:myApp/schema/user.ex`.

For a user, we have to make a couple of considerations.

* We want to store the email
* storing the password is a nono!! we should store the password hash

```elixir
defmodule Myapp.Schema.User do
  use Ecto.Schema
  import Ecto
  import Ecto.Changeset
  import Ecto.Query

  schema "users" do
    field :email, :string
    field :password_hash, :string
    field :password, :string, virtual: true
    field :password_confirmation, :string, virtual: true
    
    timestamps()
  end
end

```

> [https://hexdocs.pm/ecto/Ecto.Schema.html](https://hexdocs.pm/ecto/Ecto.Schema.html)

The first couple of import statements bring in a set of macros and methods that we can take advantage of here. I'll explain them in more detail.

* Ecto.Query - the query language combinators we can use to create query objects.
* Ecto.Changeset - methods and types for creating changesets.

Next we actually define the properties of our schema.
the `"users"` after schema tells us what table in the database to back this up to.
You may have noticed the virtual types for password and password_hash.
Its a way of declaring properties that can be part of a struct but will not be persisted to the database.
In this case, we use them to be able to create a user object and modify it using those stored properties later before saving.
The final output would then be set to password_hash

The main idea that I want to impress here is that we use this schema declaration for declaring how we are going to create out changesets.

# Changeset

A changeset is a type of elixir struct that contains information pertaining to how a database should be modified. 

By convention, we add methods for creating changesets to the schema using `Ecto.Changeset.cast\3` to cast items from a map.

```elixir
defmodule Myapp.Schema.User do
  use Ecto.Schema
  import Ecto
  import Ecto.Changeset
  import Ecto.Query

  # ...

  def changeset(struct, params \\ %{})) do
    struct
      |> cast(params, [:email, :password_hash])
  end

end
```

Myapp.Schema.User is a particular type of elixir struct that interacts with the database. We can create an empty one with %Myapp.Schema.User{}. 
This gets passed as the first argument to the cast and tells ecto what kind of database record we want to create. The third argument is a list of keys to pull from the parameters. The keys can be symbols or strings. This comes in handy for pulling body parameters from an http request or a map from an absinthe graphql router.

```elixir
alias Myapp.Schema.User
alias Myapp.Repo
user_changeset = User.create_changeset(%User{}, %{
  email: "cultofmetatron@example.com"
})

#saves to the database
{:ok, %User{}=user} = Repo.insert(user_changeset)

```

In rails or django, validations are setup at the object level and apply to all transformations to the database. Ecto takes the smarter approach of looking at the interactions themselves. It doesn't assume a validation to be universally used. Thus you create a set of changesets for different operations. A User may have a signup changeset and a update password changeset. As an admin, I may want to be able to update the password_hash directly and bypass any schema level pre-transformations.

To create signup, lets say we don't want to allow access to the password hash directly. The user should send up their email, password and password confirmation.

Lets start with this basic form

```
def signup(struct, params \\ %{}) do
  struct
    |> cast(params, [:email, :password, :password_confirmation])
end

```

Inside this changeset, we want to be able to...

  * verify that email, password and password confirmation exist
  * verify that the email is unique
  * verify that email is a valid email
  * verify that password is at least 5 characters long
  * verify that password and password confirmation are the same
  * generate a password hash and add it to the changeset.

Ecto comes with a set of validation functions we can use

  * [Ecto.Changeset.validate_required](https://hexdocs.pm/ecto/Ecto.Changeset.html#validate_required/3)
  * [Ecto.Changeset.validate_format](https://hexdocs.pm/ecto/Ecto.Changeset.html#validate_format/4)
  * [Ecto.Changeset.validate_length](https://hexdocs.pm/ecto/Ecto.Changeset.html#validate_length/3)


Each function takes a changeset and returns a changeset. If any of the validations fail, the resulting changeset will have a falsy valid? property.

```elixir

def signup(struct, params \\ %{}) do
  struct
    |> cast(params, [:email, :password, :password_confirmation])
    |> validate_required([email, :password, :password_confirmation])
    |> validate_format(:email, ~r/@/)
    |> validate_length(:password, min: 5)
end

```

Now we just need to validate that the password and password confirmation pass.
Since validations are just functions that take a changeset and return a  changeset, Its easy to add a helper method to perform the validation.

```elixir

  def signup(struct, params \\ %{}) do
    struct
      |> cast(params, [:email, :password, :password_confirmation])
      |> validate_required([email, :password, :password_confirmation])
      |> validate_format(:email, ~r/@/)
      |> validate_length(:password, min: 5)
      |> password_and_confirmation_matches()
  end


  defp password_and_confirmation_matches(changeset) do
    password = get_change(changeset, :password)
    password_confirmation = get_change(changeset, :password_confirmation)
    if password == password_confirmation do
      changeset
    else
      changeset
        |> add_error(:password_confirmation, "password_confirmation does not match password!")
    end
  end

```



`get_change` is a method on Ecto.Changeset. Notice that if the passwords don't match, we add the error to the returned changeset. In this way we can create custom validation functions.

To update the changeset with a password_hash, we can use [Comeonin](https://github.com/riverrun/comeonin)

```elixir

  def signup(struct, params \\ %{}) do
    struct
      |> cast(params, [:email, :password, :password_confirmation])
      |> validate_required([email, :password, :password_confirmation])
      |> validate_format(:email, ~r/@/)
      |> validate_length(:password, min: 5)
      |> password_and_confirmation_matches()
      |> generate_password_hash()
  end

  # ...

  defp generate_password_hash(changeset) do
    password = get_change(changeset, :password)
    hash = Comeonin.Bcrypt.hashpwsalt(password)
    changeset |> put_change(:password_hash, hash)
  end

```

You may be looking for a `validate_uniqueness` methods. You won't find it. Instead, Ecto assumes you have set a uniqueness constraint on your database in your migration.

```
defmodule Myapp.Web.Repo.Migrations.Schema.User do
  use Ecto.Migration

  def change do
    create unique_index(:users, [:email])
  end
end

```

By default, if the Repo encounters an error updating the database, it will send back a full exception. This follows Elixir's fail fast for unforseen errors. In this case, we want to be able to send meaningful feedback that the input is invalid. For that we have [unique_constraint](https://hexdocs.pm/ecto/Ecto.Changeset.html#unique_constraint/3)

```elixir

  def signup(struct, params \\ %{}) do
    struct
      |> cast(params, [:email, :password, :password_confirmation])
      |> unique_constraint(:email, message: "that email is already taken")
      |> validate_required([email, :password, :password_confirmation])
      |> validate_format(:email, ~r/@/)
      |> validate_length(:password, min: 5)
      |> password_and_confirmation_matches()
      |> generate_password_hash()
  end

```

## Testing

The biggest aha moment for me as I transitioned to elixir was just how easy it was to test the code was.
By default mix projects come with fantastic test support built in and phoenix adds a really nice test harness for running your schema validations against a database.

```elixir
defmodule Myapp.Test.Schema.UserTest do
  use Myapp.DataCase
  alias Myapp.Schema.User


  test "rejects if passwords don't match" do
    bad_pass = %{
        email: "brad@example.com",
        password: "asfga67585ASDF",
        password_confirmation: "asfga67585AS"
      }
    changeset = User.signup_changeset(%User{}, bad_pass)
    refute changeset.valid?
  end


end
```





