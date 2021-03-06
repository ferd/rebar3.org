---
title: "From Rebar 2.x to Rebar3"
excerpt: ""
---
#  From Rebar 2.x  to Rebar3


Rebar3 changes quite a few things from rebar 2.x. By and far, rebar3 attempts to keep full compatibility when handling OTP applications written in pure Erlang. People who built or wrote standard OTP apps will therefore see very few incompatibilities, although the ground still needs to be prepared.



Major changes include (but are not limited to) moving all build artifacts to `_build`, and changing the way dependencies are fetched.



## Pure Erlang, Standard OTP Applications



The first step to converting your application is to remove old directories of artifacts:



- Delete `ebin/`. Rebar3 will read and copy `ebin/` directories that exist to its own `_build/` subdirectory, so that `.app` and `.beam` files generated by other tools remain usable.

  If you used rebar 2.x, `ebin/` is where it'd put its artifacts, and those will conflict with rebar3

- remove `_rel/` directory if you used `relx`. Relx is now integrated to rebar3 and will store its artifacts in `_build`.

- remove `.rebar` directory. It is no longer used

- remove the `deps/` directory; dependencies now go in `_build` and this can prevent conflicts.

- remove these entries from `.gitignore`, `.hgignore`, or whatever



Once these directories are cleaned, the `rebar.config` configuration file can be stripped a bit: 



- remove `sub_dirs` and `lib_dirs` entries, they likely aren't necessary anymore; rebar3 automatically recognizes `src/`, `apps/*` and `lib/*` directories holding Erlang source files and OTP applications.



You should then review your `.app.src` files to make sure that the following is respected:



 - applications you depend on are listed in the `applications` tuple

 - no circular dependencies

 - the following fields are mentioned (to work with releases!)

 - `description`, `vsn`, `registered`, `applications`, `env`

 

## Required Directory Structure



Rebar3 explicitly handles releases and OTP applications only. Dependencies should only be OTP applications, one per entry.



Make sure you have one of the following directory structures:



```

src/*.{erl,app.src}

```



Which is a single-directory, single app structure. It is suitable for OTP applications, OTP releases, and escripts with a single top-level application. The `src` directory is searched recursively and the files are handles as if they were all at the top level.



The other form is by using an 'umbrella' structure:



    apps/app1/src/*.{erl,app.src}

    apps/app2/src/*.{erl,app.src}

    apps/app3/src/*.{erl,app.src}



or



    lib/app1/src/*.{erl,app.src}

    lib/app2/src/*.{erl,app.src}

    lib/app3/src/*.{erl,app.src}



That format will hold many OTP applications at once and be amenable only to releases and escripts, not OTP libraries that can be used as dependencies.



## Dependency Handling



Rebar3 now supports Hex packages and profiles. As such, consider:



- moving your dependencies to packages, see

  [Dependencies](doc:dependencies),  and [Publishing Packages](doc:publishing-packages) 

- moving your dependencies like `meck` and `PropEr` to the `test` [profile](doc:profiles)

  to get them out of your general build. It's likely your deps still include it so it may remain in the default builds too, sadly.

- you no longer need to tell rebar3 to fetch deps, it just knows whenever you tell it to compile or run tests.



Also note that Rebar3 no longer re-compiles dependencies once it has done so before; use [`_checkout` dependencies](http://www.rebar3.org/docs/dependencies#checkout-dependencies) if you want that behaviour back. 



Rebar3 also no longer checks or enforces dependency versions, and instead uses a 'nearest to the root' dependency selection algorithm in case of conflicts. The details are in [Dependencies](doc:dependencies).



With this said and with your project ready, any rebar3 guides in the documentation should be understandable and applicable.



## Other Gotchas and Compilers



By default, Rebar3 sticks to the compilers available to `erlc`: erlang, yecc, MIBs, and so on. If you have:



- **C code** you're going to need to [move things to makefiles](doc:building-cc)  or use the [port compiler plugin](http://www.rebar3.org/docs/using-available-plugins#port-compiler) (backwards compatible with rebar 2.x)

- if you used quickcheck or proper, you have to [use the plugin for that](http://www.rebar3.org/docs/using-available-plugins#quickcheck)

- [**diameter**](http://www.rebar3.org/docs/using-available-plugins#diameter) has its own plugin

- [**erlydtl**](http://www.rebar3.org/docs/using-available-plugins#erlydtl) has its own plugin

- you will have problems building some libraries with weird build tool interactions, specifically `edown` and similar libraries. In case of problems with these, heading to the #rebar channel on IRC will have community members point you to the easiest workaround.

- **reltool** releases are no longer supported, and instead, Relx is used and documented in [Releases](doc:releases) 



## Others



- if you are still using makefiles to create shortcut commands, consider using [aliases](http://www.rebar3.org/docs/using-available-plugins#alias)

- for code coverage, you will want to use the 'cover' command, as in 'rebar3 do eunit, cover'. See the [cover](http://www.rebar3.org/docs/commands#cover) documentation for more details.



## Maintaining backwards compatibility while using Hex packages



If you want to use rebar3 config and options most of the time and provide backwards compatibility to rebar2 users, add something similar to following contents in [`rebar.config.script`](doc:dynamic-configuration), which will let older rebar versions dynamically ditch hex packages and give them the opportunity to include material that is now part of profiles in rebar3:

	 case erlang:function_exported(rebar3, main, 1) of
	    true -> % rebar3
	        CONFIG;
	    false -> % rebar 2.x or older
	        %% Rebuild deps, possibly including those that have been moved to
	        %% profiles
	        [{deps, [
	            {my_dep, "VSN", {git, "https://...", {tag, "VSN"}}},
	            ...
	        ]} | lists:keydelete(deps, 1, CONFIG)]
	end.
	 
