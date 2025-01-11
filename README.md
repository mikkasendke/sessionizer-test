# sessionizer.wezterm
A sessionizer plugin for WezTerm inspired by a discussion started by [@keturiosakys](https://github.com/keturiosakys) over at https://github.com/wez/wezterm/discussions/4796 and originally inspired by ThePrimeagen's tmux-sessionizer. It helps you switch between (by default) WezTerm workspaces more easily.

## Optional dependencies (recommended)
* There is a built-in `generator` that uses [`fd`](https://github.com/sharkdp/fd)

> [!NOTE]
> A `generator` is a function that _generates_ options for you to choose from

## Installation
To install `sessionizer.wezterm`, add the following two lines __after__ `config.keys` to your wezterm.lua
```lua
local sessionizer = wezterm.plugin.require "https://github.com/mikkasendke/sessionizer.wezterm"
sessionizer.apply_to_config(config)
```
You now have the following two keybinds, custom binds are expained further down.
 * `ALT+s` show sessionizer
 * `ALT+m` switch to the most recently selected workspace

But when you press `ALT+s` the list of options is still empty. Let's fix that!

We call a table of options we can choose from a `spec`.

By default `ALT+s` shows the sessionizer for the spec assigned to `sessionizer.spec` which right now is an empty table, so we set `sessionizer.spec` to something different.
```lua
sessionizer.spec = {
    sessionizer.builtin.DefaultWorkspace {},
    -- The things that go here produce entries an Entry has a label and an id, by default the id is assumed to be a path.
    { label = "This is my home directory", id = wezterm.home_dir },
    -- You can also just put a string and the label will be the same as the path.
    "/home/mikka", -- this gives us { label = "/home/mikka", id = "/home/mikka" }

    -- You can also put so called generator functions here they return a table of entries
    -- There are some built-in generators like the return value of sessionizer.FdSearch
    sessionizer.builtin.FdSearch "/home/mikka/dev", -- This will search for git repositories in the specified directory
    -- But you can also put your own generator functions
    function()
        local entries = {}
        for i = 1, 10, 1 do
            table.insert(entries, { label = "Stub #" .. i, id = i } -- Note that i as the path for the workspace won't work 
        end
    end,
    sessionizer.builtin.Zoxide, -- this exists too
}
```
This is a basic example, there are more things you can do with a spec, we will explore them properly further down.

A spec can contain:
* An Entry (like above this is a table with a label and an id)
* A string (this will be a Entry with label and id set to the string)
* Another spec (good for grouping processing/styiling)
* A generator which is a function that returns a spec
* A name which is a string that can be used to find a spec inside another spec
* options which is a table that sets things like the title and description etc.
* processors which is a function or table of functions that takes a table of entries and can modify them

## Customization
### Styling
A spec is best styled by using processors and `wezterm.format`. A processor takes an array/table of Entry and a function calling the next processor.
If you need just one processor use `<any spec>.processor` if you need multiple you can put them into `<any spec>.processors` which is a table of processors.
The entry array you will get contains all entries generated by the spec you are in minus the entries a processor might have removed before.
For example:

```lua
sessionizer.spec = {
    {
        sessionizer.builtin.AllActiveWorkspaces { show_current_workspace = true, show_default_workspace = false, },
        processors = {
            function(entries) -- this is a bit annoying if you don't need the context of other entries so just
                              -- there is sessionizer.helpers.for_each_entry which is shown below
                for _, entry in pairs(entries) do
                    entry.label = wezterm.format {
                        { Foreground = { Color = "#77ee88" } },
                        { Text = entry.label }
                    }
                end
            end,
            sessionizer.helpers.for_each_entry(function(entry) entry.label = "active: " .. entry.label end),
        },
    },
    sessionizer.builtin.Zoxide,
    processor = sessionizer.helpers.for_each_entry(function(entry) entry.label = entry.label:gsub(wezterm.home_dir, "~") end)
}
```
### Key binds
You can disable the default bindings by passing an additional true to the `apply_to_config` function like so
```lua
sessionizer.apply_to_config(config, true)
```
