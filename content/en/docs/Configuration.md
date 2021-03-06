---
title: "Configuration"
excerpt: ""
---
#  Configuration


## Global (all-commands) configuration

Rebar3 supports some options that will impact the behaviour of the tool wholesale. Those are defined as OS environment variables, as follows:



	 REBAR_PROFILE="term"         # force a base profile
	HEX_CDN="https://..."        # change the Hex endpoint for a private one
	REBAR_CONFIG="rebar3.config" # changes the name of rebar.config files
	QUIET=1                      # only display errors
	DEBUG=1                      # show debug output
	                             # "QUIET=1 DEBUG=1" displays both errors and warnings
	REBAR_COLOR="low"            # reduces amount of color in output if supported
	REBAR_CACHE_DIR              # override where rebar3 stores cache data
	REBAR_GLOBAL_CONFIG_DIR      # override where rebar3 stores config data
	http_proxy                   # standard proxy ENV variable is respected
	https_proxy                  # standard proxy ENV variable is respected


## Alias

Aliases allow to create new commands out of existing ones, as if they were run one after the other:

	 {alias, [{check, [eunit, {ct, "--sys_config=config/app.config"}]}]}.
Arguments (as with a command line) can be passed by replacing `Provider` with `{Provider, Args}`.

## Artifacts

Artifacts are a list of files that are required to exist after a successful compilation (including any compile hooks) other than Erlang modules. This is useful to let `rebar3` figure out if a dependency with some non-Erlang artifacts was successfully built or not. Examples where this can be useful include identifying C code built for shared libraries, rendered templates, or that an escript was properly generated.



If a dependency is found to already be built (meaning its `.app` file's list of modules matches its `.beam` files and all its artifacts exist) it will not have the compilation provider or its hooks called on it during a subsequent run of `rebar3`.

	 {artifacts, [file:filename_all()]}.
What the paths are relative to depends on if it is defined at the top level of an umbrella project or not. For example, let's say we have a project `my_project` that contains an application `my_app` under `apps/my_app/` that we build an `escript` from. Then we want to tell `rebar3` to not consider the configuration in `my_project/rebar.config`, since it is the top level `rebar.config` of an umbrella. The `artifact` will be relative to the `profile_dir`, which by default is `_build/default/`:

	 {escript_name, rebar3}.

	{provider_hooks, [{post, [{compile, escriptize}]}]}.

	{artifacts, ["bin/rebar3"]}.
If instead it isn't an umbrella but with `my_app` being the top level directory the artifact defined in `rebar.config` is relative to the application's output directory, in this case `_build/default/lib/my_app/` but we can use a template to define it from `profile_dir` as well:

	 {escript_name, rebar3}.

	{provider_hooks, [{post, [{compile, escriptize}]}]}.

	{artifacts, ["{{profile_dir}}/bin/rebar3"]}.
All available template key are listed in the table below.

| Template Key | Description                                                                             |
| ------------ | --------------------------------------------------------------------------------------- |
| profile_dir  | The base output directory with the profile string appended, default: `_build/default/`. |
| base_dir     | The base output directory, default: `_build`.                                           |
| out_dir      | The application's output directory, default: `_build/default/lib//`.       |


One more example would be using artifacts within an override, in this case for `eleveldb`:

	 {overrides,
	 [{override, eleveldb, [{artifacts, ["priv/eleveldb.so"]},
	                        ...
	                       ]
	  }]}.
The artifact is define on the application `eleveldb` so it is relative to the output directory, meaning the path used above is the same as if we used `"{{out_dir}}/priv/eleveldb.so"`.

## Compilation

Compiler options can be set with `erl_opts`, available options are listed in the Erlang compile module's [documentation](http://www.erlang.org/doc/man/compile.html).

	 {erl_opts, []}.

Additionally it is possible to set platform specific options. The options run a regular expression against the OTP version joined with the OS details.

	  {erl_opts, [{platform_define,
	               "(linux|solaris|freebsd|darwin)",
	               'HAVE_SENDFILE'},
	              {platform_define, "(linux|freebsd)",
	                'BACKLOG', 128},
	              {platform_define, "^18",
	                'OTP_GREATER_THAN_18'},
	              {platform_define, "^R13",
	                'old_inets'}]
	}.
The version string might look like `"22.0-x86_64-apple-darwin18.5.0-64"` so if you want to check for a specific OTP version (and avoid matching against an OS version), remember to prefix your regular expression with `^`.

A separate compile option is one to declare modules to compile before all others:

	 {erl_first_files, ["src/mymodule.erl", "src/mymodule.erl"]}.

And some other general options exist:

	 {validate_app_modules, true}. % Make sure modules in .app match those found in code
	{app_vars_file, undefined | Path}. % file containing elements to put in all generated app files
	%% Paths the compiler outputs when reporting warnings or errors
	%% relative (default), build (all paths are in _build, default prior
	%% to 3.2.0, and absolute are valid options
	{compiler_source_format, relative}.
Other Erlang-related compilers are supported with their own configuration options:

- [Leex compiler](http://erlang.org/doc/man/leex.html) with `{xrl_opts, [...]}`

- [SNMP MIB Compiler](http://www.erlang.org/doc/apps/snmp/snmp_mib_compiler.html) with `{mib_opts, [...]}`

- [Yecc compiler](http://erlang.org/doc/man/yecc.html) with `{yrl_opts, [...]}`

### rebar3 compiler options

rebar3 ships with some compiler options specific to rebar3.

#### Enable/Disable recursive compiling

Disable or enable recusrive compiling globally

```
{erlc_compiler,[{recursive,boolean()}]}.
```

Disable or enable recursive compiling on src_dirs:

```
{erl_opts, [{src_dirs,[{string(),[{recursive,boolean()}]}]}]}.
```

Disable or enable recursive compiling on for extra src dirs:

```
{erl_opts, [{extra_src_dirs,[{string(),[{recursive,boolean()}]}]}]}.
```

##### Examples

All three options can be combined for granularity which should provide enough
flexibility for any project. Below are some examples:

Disable recursive compiling globally, but enable it for a few dirs:

```
{erlc_compiler,[{recursive,false}]},
{erl_opts,[{src_dirs,["src",{"other_src",[{recursive,true}]}]}]}.
```

Disable recursive compiling on test and other dirs:

```
{erl_opts, [
            {extra_src_dirs,[
                    {"test", [{recursive,boolean()}]},
                    {"other_dir", [{recursive,boolean()}]}]}
            ]
}.
```

## Common Test

	 {ct_first_files, [...]}. % {erl_first_files, ...} but for CT
	{ct_opts, [...]}. % same as options for ct:run_test(...)
	{ct_readable, true | false}. % disable rebar3 modifying CT output in the shell
Reference of common test options for `ct_opts`: [http://www.erlang.org/doc/man/ct.html#run_test-1](http://www.erlang.org/doc/man/ct.html#run_test-1)

A special option allows to load a default `sys.config` set of entries using `{ct_opts, [{sys_config, ["name.of.config"]}]}`

Options often exist mirroring those that can be specified in [Commands](doc:commands) arguments.

## Cover

Enable code coverage in [tests](doc:running-tests) with `{cover_enabled, true}`. Then the `cover` provider can be run after tests to show reports. The option `{cover_opts, [verbose]}` can be used to force coverage reports be printed to the terminal rather than just in files. Specific modules can be blacklisted from code coverage by adding `{cover_excl_mods, [Modules]}` to the config file. Applications can be blacklisted as a whole with the `{cover_excl_apps, [AppNames]}` option.

## Dialyzer



	 -type warning() :: no_return | no_unused | no_improper_lists | no_fun_app | no_match | no_opaque | no_fail_call | no_contracts | no_behaviours | no_undefined_callbacks | unmatched_returns | error_handling | race_conditions | overspecs | underspecs | specdiffs

	{dialyzer, [{warnings, [warning()]},
	            {get_warnings, boolean()},
	            {plt_apps, top_level_deps | all_deps} % default: top_level_deps
	            {plt_extra_apps, [atom()]},
	            {plt_location, local | file:filename()},
	            {plt_prefix, string()},
	            {base_plt_apps, [atom(), ...]},
	            {base_plt_location, global | file:filename()},
	            {base_plt_prefix, string()}]}.
For information on suppressing warnings in modules see the [Requesting or Suppressing Warnings in Source Files](http://erlang.org/doc/man/dialyzer.html) section of the Dialyzer documentation.

## Distribution

Multiple providers and plugins may demand to support distributed Erlang. Generally, the configuration for all such commands (such as `ct` and `shell`) respect the following configuration values:

	 {dist_node, [
	    {setcookie, 'atom-cookie'},
	    {name | sname, 'nodename'},
	]}.


## Directories

The following options exist for directories; the value chosen below is the default value

	 %% directory for artifacts produced by rebar3
	{base_dir, "_build"}.
	%% directory in '<base_dir>/<profile>/' where deps go
	{deps_dir, "lib"}.
	%% where rebar3 operates from; defaults to the current working directory
	{root_dir, "."}.
	%% where checkout dependencies are to be located
	{checkouts_dir, "_checkouts"}.
	%% directory in '<base_dir>/<profile>/' where plugins go
	{plugins_dir, "plugins"}.
	%% directories where OTP applications for the project can be located
	{project_app_dirs, ["apps/*", "lib/*", "."]}.
	%% Directories where source files for an OTP application can be found
	{src_dirs, ["src"]}.
	%% Paths to miscellaneous Erlang files to compile for an app
	%% without including them in its modules list
	{extra_src_dirs, []}.
	%% Paths the compiler outputs when reporting warnings or errors
	%% relative (default), build (all paths are in _build, default prior
	%% to 3.2.0, and absolute are valid options
	{compiler_source_format, relative}.
Furthermore, rebar3 stores some of its configuration data in `~/.config/rebar3` and cache some data in `~/.cache/rebar3`. Both can be overridden by specifying `{global_rebar_dir, "./some/path"}.`

## EDoc

All options supported by [EDoc](http://www.erlang.org/doc/man/edoc.html#run-3) can be put in `{edoc_opts, [...]}`.

## Escript

Full details at the [escriptize command](http://www.rebar3.org/v3/docs/commands#escriptize). Example configuration values below.

	 {escript_main_app, AppName}. % specify which app is the escript app
	{escript_name, "FinalName"}. % name of final generated escript
	{escript_incl_apps, [App]}. % apps (other than the main one and its deps) to be included
	{escript_emu_args, "%%! -escript main Module\n"}. % emulator args
	{escript_shebang, "#!/usr/bin/env escript\n"}. % executable line
	{escript_comment, "%%\n"}. % comment at top of escript file

Because of the structure of escript building, options at the top-level rebar.config file only are used to build an escript.

## EUnit



	 {eunit_first_files, [...]}. % {erl_first_files, ...} but for CT
	{eunit_opts, [...]}. % same as options for eunit:test(Tests, ...)
	{eunit_tests, [...]}. % same as Tests argument in eunit:test(Tests, ...)
Eunit Options reference: [http://www.erlang.org/doc/man/eunit.html#test-2](http://www.erlang.org/doc/man/eunit.html#test-2)

## Hex Repos and Indexes

Starting with rebar3 version 3.7.0, multiple Hex repositories (or indexes) can be used at the same time. Repositories are declared in an ordered list, from highest priority to lowest priority.



When looking for a package, repositories are going to be traversed in order. As soon as one of the packages fits the description, it is downloaded. The hashes for each found packages are kept in the project's lockfile, so that if the order of repositories changes and some of them end up containing conflicting packages definitions for the same name and version pairs, only the expected one will be downloaded.



This allows the same mechanism to be used for both mirrors, private repositories (as provided by hex.pm), and self-hosted indexes.



For publishing or using a private repository you must use the [rebar3_hex](https://github.com/tsloughter/rebar3_hex) plugin to authenticate, `rebar3 hex auth`. This creates a separate config file `~/.config/rebar3/hex.config` storing the keys.

	 {hex, [
	   {repos, [
	      %% A self-hosted repository that allows publishing may look like this
	      #{name => <<"my_hexpm">>,
	        api_url => <<"https://localhost:8080/api">>,
	        repo_url => <<"https://localhost:8080/repo">>,
	        repo_public_key => <<"-----BEGIN PUBLIC KEY-----
	        ...
	        -----END PUBLIC KEY-----">>
	      },
	      %% A mirror looks like a standard repo definition, but uses the same
	      %% public key as hex itself. Note that the API URL is not required
	      %% if all you do is fetch information
	      #{name => <<"jsDelivr">>,
	        repo_url => <<"https://cdn.jsdelivr.net/hex">>,
	        ...
	       },
	       %% If you are a paying hex.pm user with a private organisation, your
	       %% private repository can be declared as:
	       #{name => <<"hexpm:private_repo">>}
	       %% and authenticate with the hex plugin, rebar3 hex user auth
	   ]}
	]}.

	%% The default Hex config is always implicitly present.
	%% You could however replace it wholesale by using a 'replace' value,
	%% which in this case would redirect to a local index with no signature
	%% validation being done. Any repository can be replaced.
	{hex, [
	   {repos, replace, [
	      #{name => <<"hexpm">>,
	        api_url => <<"https://localhost:8080/api">>,
	        repo_url => <<"https://localhost:8080/repo">>,
	        ...
	       }
	   ]}
	]}.



## Minimum OTP Version

A minimum version of Erlang/OTP can be specified which causes a build to fail if an earlier version is being used to build the application.

	 {minimum_otp_vsn, "17.4"}.


## Overrides

Overrides allow to modify the configuration of a dependency from a higher level application. They are meant to allow quick fixes and workarounds, although we do recommend working on permanent fixes that make it to the target app's configuration when possible.



Overrides come in 3 flavours: add, override on app and override on all.

	 {overrides, [{add, app_name(), [{atom(), any()}]},
	             {del, app_name(), [{atom(), any()}]},
	             {override, app_name(), [{atom(), any()}]},
	             {add, [{atom(), any()}]},
	             {del, [{atom(), any()}]},
	             {override, [{atom(), any()}]}]}.
These are applied to dependencies, and dependencies can have their own overrides as well that are applied. in the order overrides on all, per app overrides, per app additions.



As an example, this can be used to force all the dependencies to be compiled with `debug_info` by default, and to force `no_debug_info` in case the production profile is used.

	 {overrides, [{override, [{erl_opts, [debug_info]}]}]}.

	{profiles, [{prod, [{overrides, [{override, [{erl_opts,[no_debug_info]}]}]},
	                    {relx, [{dev_mode, false},
	                            {include_erts, true}]}]}
	           ]}.
Another example could be to remove `warnings_as_errors` as a compiler option for all applications:

	 {overrides, [
	    %% For all apps:
	    {del, [{erl_opts, [warnings_as_errors]}]},
	    %% Or for just one app:
	    {del, one_app, [{erl_opts, [warnings_as_errors]}]}
	]}.


## ! Overrides For All Apps in Umbrella Projects !

	 In an umbrella project, overrides that are specified in the top-level rebar.config file will also apply to applications within the apps/ or lib/ directory. By comparison, overrides specified in the rebar.config file at the application level will only apply to their dependencies.



## Hooks

There are two types of hooks: shell hooks and provider hooks. They both apply [to the same type of providers](#hookable-providers).



## Shell Hooks



Hooks provide a way to run arbitrary shell commands before or after hookable providers, optionally first matching on the type of system to choose which hook to run. Shell hooks are run after provider hooks.

	 -type hook() :: {atom(), string()}
	              | {string(), atom(), string()}.

	{pre_hooks, [hook()]}.
	{post_hooks, [hook()]}.
An example for building [merl](https://github.com/richcarl/merl) with rebar3 by using `pre_hooks`:

	 {pre_hooks, [{"(linux|darwin|solaris)", compile, "make -C \"$REBAR_DEPS_DIR/merl\" all -W test"},
	             {"(freebsd|netbsd|openbsd)", compile, "gmake -C \"$REBAR_DEPS_DIR/merl\" all"},
	             {"win32", compile, "make -C \"%REBAR_DEPS_DIR%/merl\" all -W test"},
	             {eunit, "erlc -I include/erlydtl_preparser.hrl -o test test/erlydtl_extension_testparser.yrl"},
	             {"(linux|darwin|solaris)", eunit, "make -C \"$REBAR_DEPS_DIR/merl\" test"},
	             {"(freebsd|netbsd|openbsd)", eunit, "gmake -C \"$REBAR_DEPS_DIR/merl\" test"},
	             {"win32", eunit, "make -C \"%REBAR_DEPS_DIR%/merl\" test"}
	            ]}.


## ! Behaviour of post_hooks !

	 A `post_hooks` entry will only be called if its hookable provider was successful. This means that if you add a `post_hooks` entry for `eunit`, it will only be called if your EUnit tests are able to finish successfully.

## Provider Hooks



Providers are also able to be used as hooks. The following hook runs `clean` before `compile` runs. To execute commands in a namespace a tuple is used as second argument. Provider hooks are run before shell hooks.

	 {provider_hooks, [{pre, [{compile, clean}]}
	                  {post, [{compile, {erlydtl, compile}}]}]}


## Hookable Points in Providers



Only specific built-in providers support hooks attached to them. Control depends on whether the provider operates on the project's applications (each application and dependency) or if it's expected to only run on the project at large.



Provider hooks are run before shell hooks.

| Hook         | before and after                                                                                    |
| ------------ | --------------------------------------------------------------------------------------------------- |
| clean        | each application and dependency,  and/or before and after all top-level applications are compiled\* |
| ct           | the entire run                                                                                      |
| compile      | each application and dependency, and/or before and after all top-level applications are compiled\*  |
| edoc         | the entire run                                                                                      |
| escriptize   | the entire run                                                                                      |
| eunit        | the entire run                                                                                      |
| release      | the entire run                                                                                      |
| tar          | the entire run                                                                                      |
| erlc_compile | compilation of the beam files for an app                                                            |
| app_compile  | building of the .app file from .app.src for an app                                                  |


\* These hooks are, by default, running for every application, because dependencies may specify their own hook in their own context. The distinction is that in some cases (umbrella apps), hooks can be defined on many levels (omitting overrides):



- the rebar.config file at the application root

- each top-level app's (in `apps/` or `libs/`) rebar.config

- each dependency's rebar.config



By default, when there is no umbrella app, the hook defined in the top-level rebar.config is attributed to be part of the top-level app. This allows the hook to keep working for a dependency when the library is later published.



If however the hook is defined in rebar.config at the root of a project with umbrella applications, the hooks will be run before/after the task runs for *all* of the top-level applications.



To preserve the per-app behaviour in an umbrella project, hooks must instead be defined within each application's rebar.config.

## Relx

See [Releases](doc:releases)

## Shell

The `rebar3 shell` REPL will automatically boot applications if a `relx` entry is found, but apps to be started by the shell can be specified explicitly with `{shell, [{apps, [App]}]}`.



Other options include:

| Option               | Value                    | Description                                                                                                                                                                  |
| -------------------- | ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| apps                 | [app1, app2, ...]        | Applications to be booted by the shell. Overtakes the `relx` entry values                                                                                                    |
| config               | "path/to/a/file.config"  | Loads a `.config` file (such as `sys.config`) for the applications to be booted by the shell.                                                                                |
| script_file          | "path/to/a/file.escript" | Evaluates a given escript before booting the applications for a node.                                                                                                        |
| app_reload_blacklist | [app1, app2, ...]        | Applications that should not be reloaded when calling commands such as `r3:compile()`. Useful when working with applications such as `ranch`, which crash after two reloads. |

## XRef



	 {xref_warnings,false}.
	{xref_extra_paths,[]}.
	{xref_checks,[undefined_function_calls,undefined_functions,locals_not_used,
	              exports_not_used,deprecated_function_calls,
	              deprecated_functions]}.
	{xref_queries,[{"(xc - uc) || (xu - x - b - (\"mod\":\".*foo\"/\"4\"))", []}]}.
	{xref_ignores, [Module, {Module, Fun}, {Module, Fun, Arity}]}.
