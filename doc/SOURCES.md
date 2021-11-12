# Sources

This document describes the source API, which allows users to register /
deregister sources and get registered sources.

All methods are available on the main `null-ls` module unless specified
otherwise. Methods not mentioned here are either internal or unstable, so use
them at your own risk.

```lua
require("null-ls").get_sources()
```

Methods that add, remove, or otherwise change sources prompt nvim-lspconfig to
update its list of filetypes, as displayed in `:LspConfig`.

## get_sources()

Returns a list (array-like table) of all registered sources. The returned table
references the same table that null-ls uses internally, so mutating sources will
affect how null-ls operates accordingly.

Registered sources have the following structure, which differs from their
pre-registration structure:

```lua
local example_source = {
    name = "example_source",
    filetypes = { ["lua"] = true },
    methods = { [require("null-ls").methods.FORMATTING] = true },
    generator = {
        fn = function()
            return "I am a source!"
        end,
    },
    id = 1
}
```

### name

The name of the source. Sources without a name are automatically assigned the
name `anonymous source`. Users or integrations may register any number of
sources with the same name.

### filetypes

A table of filetypes, where each key represents a filetype and each value
indicates whether the source supports that filetype. Transformed from the
original `filetypes` and `disabled_filetype` lists.

The special key `_all` indicates that a source is active for all filetypes
(unless superseded by a `false` value for a filetype key).

### methods

A table of methods, where each key represents a supported method. Transformed
from the original method or list of methods.

### generator

The source's generator.

### id

The source's ID. Assigned automatically. Each source receives the next available
ID.

## register(to_register)

The main method for registering sources. `to_register` can have the following
structures:

- A single source (registered individually):

```lua
require("null-ls").register(my_source)
```

- A list (array-like table) of sources (registered sequentially):

```lua
require("null-ls").register({ my_source, my_other_source })
```

- A table of sources with shared configuration (`name` and `filetypes` override
  source-specific options):

```lua
require("null-ls").register({
    name = "my_sources",
    filetypes = { "lua" },
    sources = { my_source, my_other_source },
})
```

For information on sources, see [MAIN](MAIN.md).

## deregister(query)

Removes all sources matching `query` from the internal list of sources. `query`
can be a string, in which case it's treated as a name, or an object with the
following structure:

```lua
local query = {
    name = "my-source", -- string
    method = require("null-ls").methods.FORMATTING, -- null-ls method
    id = 1, -- number
}
```

All keys in the query are optional, and passing an empty query will deregister
all sources.

Note that special characters are automatically escaped when `query` is a string
but not when it's an object, which allows using Lua string matchers.

## reset_sources()

Clears all registered sources.

## is_registered(name)

Returns `true` if null-ls has registered a source with the name `name` (exact
match). For more flexible queries, check the results of `get_sources()` manually.

## register_name(name)

Allows integrations to register a name (independent of its sources), which they
can check with `is_registered(name)` to avoid double registration.