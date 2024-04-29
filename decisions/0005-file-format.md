# File Format

- Status: **Accepted**
- Deciders: Vlad Ionica, Dan Nixon, Adam Washington, Tristan Youngs
- Date:  11-03-2022

## Context of the problem

The project to date has used a homegrown file format.  While fit for
purpose, it does add overhead for managing the parsing and writing.
Additionally, as it is a custom format, there exists no tooling for
managing and editing these files.

Moving to an industry standard file format would enable us to farm out
parsing and serialisation to a library while also unlocking a
community of tooling and support.

## Decision Drivers

- The file format must be able to represent the entire contents of a Dissolve input file.
- A high quality, stable library for the file format must be available.
- Any library should prefer modern C++ over pointer passing semantics.
- The format must be amenable to human editing and adding comments.
- An error in hand editing a file should lead to a failed parse and not an incorrect input.
- Will comments persist after editing in Dissolve?
- Can common text editors provide highlighting support for this format
- Tools should be available for querying and transforming these files.
- It should be possible to update the Dissolve schema without invalidating all files in the old format.
- Is it possible to simultaneously support multiple schema versions?
- Is the format reasonably efficient.

## Considered Options

1. XML
2. JSON
3. YAML
4. TOML
5. Dhall

## Decision Outcome

Chosen option: TOML.  It provided better editing support than XML or
JSON, while having a less ambiguous syntax than YAML (forcing broken
files into failed parses instead of faulty simulations).  Dhall was
ruled out almost immediately due to poor library support.

Toml11 was chosen for the parsing library.  The API was intuitive and
was similar enough to the API of the top YAML library that it would be
possible to quickly switch out implementations, if a problem with the
library or format was found.

## Positive Consequences

- The `LineParser` class and most of our IO code can be eliminated,
  with this functionality being moved into the library.

- Common text editors (e.g. Notepad++, Vim) support syntax
  highlighting for our input files.

- Common tools (e.g. yq) enable automatically updating and query the files.

## Negative Consequences.

- We have an additional library dependency
- We must spend at least one release supporting *both* the old format
  and TOML to enable users to update their files to the new format.
- Not being a bespoke solution, TOML is inherently more verbose than
  the custom format.

## Pros and Cons of the Options

### XML

As the oldest of the formats, XML had the advantage on tooling and
library support.  Additionally, we already use an XML file for
importing certain Forcefield types, so there would be no need for an
additional dependency.  However, hand editing proved frustrating and
verbose.  Additionally, it left a large ambiguity in the design space
that could lead to erroneous, but syntactically correct, files.
Specifically, there is a natural tension between node attributes and
node children.  The attributes are more succinct for editing, but the
children are more flexible.  Incorrectly adding a child instead of an
attribute, or vice versa, would lead to errors that could be quite
subtle.

## JSON

JSON had library and tooling support that might even surpass XML.
However, our initial pass at producing JSON example files created
examples almost twice as long as any other format.  Additionally, JSON
does not have any support for comments.  These factors ruled it out
from the perspective of hand editing.

## YAML

Nearly every advantage that applies to TOML also applies to YAML.  It
even holds a slight advantage at hand editing.  However, it's
flexibility at editing proved to be more of a disadvantage.  A user
unknowingly mixing tabs and spaces could unwittingly create a file
that parses but does not perform the desired calculation.  The
ambiguity can also lead to different parsers having alternate
interpretations of a file.  For example

```
America: US
Canada: CA
Norway: NO
Germany: DE
enable: false
```

Under certain parsers, the values of `enable` and `Norway` will be
equal, as `NO` is recommended under certain profiles to be parsed as a
boolean.  Other parsers will treat it as the string `NO`.  Every tool
that the file passes through has a chance to change the file,
depending on the interpretation.

## Dhall

Dhall has some included scripting ability that could have been useful.
Unfortunately, it does not have a stable C++ library at this time, so
it was ruled out almost immediately.

## Links

- Issue
  [#899](https://github.com/disorderedmaterials/dissolve/issues/899)
  charts the initial work on this decision.
- The [toml](https://toml.io/en/) format home page
- The [toml11](https://github.com/ToruNiina/toml11) library
- The [JSON](https://www.json.org/json-en.html) format home page
- The [YAML](https://yaml.org/) format home page
- The [XML](https://www.w3.org/XML/) format home page
- The [Dhall](https://dhall-lang.org/) format home page
