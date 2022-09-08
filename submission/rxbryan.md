# Google Summer of Code 2022: Implementing support for extending MetaCall with plugins and Refactoring the MetaCall CLI

The MetaCall CLI Bootstrapping / Refactor project aims to add support for extending the MetaCall core with plugins and to also refactor the CLI to allow commands to be implemented as plugins. This has allowed for the addition of new features to the core library without making changes to the codebase.

### What was done

A new loader, the extension loader (`ext_loader`) was implemented in the core to serve as a base for implementing the plugin architecture.

#### [Refactor existing CLI commands into plugins](https://github.com/metacall/core/pull/298)

Each plugin consists of a configuration file, which is plain JSON and an arbitrary number of program files and libraries (listed in the `scripts` field of the configuration file) that may or may not export (or in the case of the extension loader, register) an arbitrary number of functions. The `language_id` field represents the _runtime loader tag_.

```JSON
{
  "language_id": "node",
  "path": ".",
  "scripts": [
    "example.js"
  ]
}
```
```JSON
{
  "language_id": "ext",
  "path": ".",
  "scripts": [
    "backtrace_plugin"
  ]
}
```
_Example plugin configuration files._

The load, eval, clear, inspect, call and await commands were refactored into plugins. The [`help`](https://github.com/metacall/core/blob/57f40a35b23d4ee0ab828efb684128474102787e/source/cli/plugins/cli_core_plugin/source/cli_core_plugin.cpp#L36) command was improved to make it more user friendly and an additional command [`get_function_list`](https://github.com/metacall/core/blob/57f40a35b23d4ee0ab828efb684128474102787e/source/cli/plugins/cli_core_plugin/source/cli_core_plugin.cpp#L697) was added to allow the list of all loaded functions to be gotten in a user-friendly manner.

#### [Add support for selectively loading plugins on startup and during runtime](https://github.com/metacall/core/pull/287)

This was implemented as a [plugin](https://github.com/metacall/core/blob/develop/source/extensions/plugin_extension/source/plugin_extension.cpp). It recursively iterates the plugin directory to load every plugin.

#### [Refactor the CLI and add support for dynamically discovering loaded commands, and verifying command arguments](https://github.com/metacall/core/pull/320)

The `execute` function was refactored to allow commands to be called both from the command line and `repl`. Support for argument size and [`type checks`](https://github.com/metacall/core/pull/330) was implemented for commands (plugins).


The `repl` has been refactored to a plugin. It is based on the NodeJs REPL API and offers editing capabilities while a user is entering the line and also a dynamic auto-complete for commands and functions (as they are loaded).

#### Refactored CLI Demo

[Demo #1](https://youtu.be/JOAK5jztPuw)

[Demo #2](https://youtu.be/uzDOPvZoNuI)

[Testing auto-complete](https://youtu.be/FktGd5pqgCY)

### Bugs fixed

- [fix segfault in metacall-ext-test #294](https://github.com/metacall/core/pull/294)
- [metacall-plugin-local-test working properly #296](https://github.com/metacall/core/pull/296)

### Bugs found

- [ext loader destroy order bug](https://github.com/metacall/core/pull/303)
- [metacall cli await bug](https://github.com/metacall/core/pull/324)
- [node_loader_impl_load_from_memory bug](https://github.com/metacall/core/pull/327)

### Open Pull Requests

- [refactor cli #320](https://github.com/metacall/core/pull/320)
- [Add metacall include path and metacall library path in c loader #270](https://github.com/metacall/core/pull/270)

### Pull Requests merged

- [add argument number and type checks in cli-core-plugin #330](https://github.com/metacall/core/pull/330)
- [test for await bug #324](https://github.com/metacall/core/pull/324)
- [fix depth in directory iterator #311](https://github.com/metacall/core/pull/311)
- [add core plugin for cli-refactor #298](https://github.com/metacall/core/pull/298)
- [metacall-plugin-local-test working properly #296](https://github.com/metacall/core/pull/296)
- [fix segfault in metacall-ext-test #294](https://github.com/metacall/core/pull/294)
- [add support for installing plugins #291](https://github.com/metacall/core/pull/291)
- [add test for load_extension #289](https://github.com/metacall/core/pull/289)
- [add extension for loading metacall*.json #287](https://github.com/metacall/core/pull/287)

### Closed Pull Requests (not merged)

- [add test for node_loader_impl_load_from_memory bug #327](https://github.com/metacall/core/pull/327)
- [fix bug in node_loader load_from_memory #325](https://github.com/metacall/core/pull/325)
- [Update plugin_extension #306](https://github.com/metacall/core/pull/306)
- [add test for double free in loader_impl.c #303](https://github.com/metacall/core/pull/303 )

#### Conclusion

In these past three months, I have been able to improve my understanding of polyglot software development, testing and debugging skills, and ability to dig through large and complex codebases. I would like to thank my mentor Vicente Eduardo Ferrer Garcia, whose thoughtful guidance helped make my GSOC experience a successful one and Gil Arasa Verge for giving me this opportunity.