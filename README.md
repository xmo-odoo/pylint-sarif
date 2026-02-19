# pylint-sarif

pylint-sarif is a [pylint](http://pylint.org) plugin which allows outputting
[SARIF][] report files.

[SARIF][] is a standardized structured interchange format used to exchange
information between static analysis and security tools (e.g. pylint) and
various types of alert systems, formatters, aggregators, ...

## Usage

    pylint --load-plugins=pylint_sarif --output=format=sarif <path to source>

or in `pyproject.toml`:

    [tool.pylint.main]
    load-plugins = ["pylint_sarif"]

then pass the `--output-format` still.

Note that pylint supports [multi-output configurations][] so it's possible
to output both a machine-readable report to a file and a human-readable
output to stdout. 

## Limitations

[SARIF][] is a massive format, consumers only implements the bits they
care about (which makes sense), but some of them may require properties
which are optional per-standard, this reporter primarily targets GitHub
with the official [SARIF validator][] being a secondary goal[^2].

Other *production* consumers should have the same status as GitHub (unless
they are kind enough to remove their extra requirements). One might hope
they would contribute their requirements as a validation rule but that
seems challenging[^1].

### On URIs

The reporter makes use of URIs for artifacts (~files). Per 
["guidance on the use of artifactLocation objects"][3.4.7], `uri` *should*
capture the deterministic part `of` the artifact location and `uriBaseId`
*should* capture the non-deterministic part. However as far as I can tell
pylint has no requirement (and no clean way to require) consistent resolution
roots: `path` is just relative to the cwd, and there is no requirement to
have project-level files to use pylint. This makes the use of relative uris
dodgy, but absolute uris are pretty much always broken for the purpose of
*interchange* so they're not really any better.

As a side-note, Github [asserts][relative-uri-guidance]

> While this [nb: `originalUriBaseIds`] is not required by GitHub for the
> code scanning results to be displayed correctly, it is required to produce
> a valid SARIF output when using relative URI references.

However per [3.4.4][] this is incorrect, the `uriBaseId` can be resolved
through end-user configuration, `originalUriBaseIds`, external information
(e.g. envvars), or heuristics.

It would be nice to document the "relative root" via `originalUriBaseIds`
(which may be omitted for that purpose per [3.14.14][], but per the above
claiming a consistent project root is dodgy.

We *could* resolve known project files (e.g. pyproject.toml, tox.ini, etc...)
in order to find a consistent root (project root, repo root, ...) and
set / use that for relative URIs but that's a lot of additional complexity
which I'm not sure is warranted.

## ???

- [pylint2sarif](https://github.com/GrammaTech/pylint-sarif) by @fishoak
  (Paul Anderson, member of the SARIF committee)
- [`Sarif.Multitool convert`][] advertises pylint json input

## TODO

- Validation test via the official validator?

  It's a bit difficult as `Sarif.Multitool` is a C# utility distributed via
  nuget, there is also a distribution via npm (`@microsoft/sarif-multitool`)
  but it's just an installation convenience, it fetches and runs the C#
  package (including a redistributable C# runtime, see
  `@microsoft/sarif-multitool-linux`)

[^1]: the validator seems like a pretty complicated C# project, the
  [Contributing a SARIF Validation Rule](https://github.com/microsoft/sarif-sdk/blob/main/docs/Contributing%20a%20SARIF%20Validation%20Rule.md)
  document is all dead links, and requests have mostly been rotting:
  right now there are 14 "New rule" issues open (8 fixed) and
  9 "RULE REQUEST" issues (3 fixed, 1 closed as duplicate), looks
  like the primary rules contributor / driver has deleted their
  account and the rest have generally moved away from the project
  (or at least reduced their activity on it)
[^2]: but it's got a bunch of weird rules which either are nonsensical
  or are not really provided-for by pylint

[SARIF]: https://sarif.info/
[SARIF validator]: https://sarif.info/Validation
[multi-output configurations]:
  https://pylint.readthedocs.io/en/stable/user_guide/usage/output.html#output-options
[`Sarif.Multitool convert`]:
  https://github.com/microsoft/sarif-sdk/blob/main/docs/multitool-usage.md
[3.4.4]:
  https://docs.oasis-open.org/sarif/sarif/v2.1.0/csprd01/sarif-v2.1.0-csprd01.html#_Toc10540869 
[3.4.7]:
  https://docs.oasis-open.org/sarif/sarif/v2.1.0/csprd01/sarif-v2.1.0-csprd01.html#_Toc10540872 
[3.14.14]:
  https://docs.oasis-open.org/sarif/sarif/v2.1.0/csprd01/sarif-v2.1.0-csprd01.html#_Toc10540936
[relative-uri-guidance]:
  https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning#relative-uri-guidance-for-sarif-producers