## Requirements gathering

* _Interoperability_: Will the program take input from other programs?
  Will its output be fed to other programs?

* _Modularity_: How large is the feature set expected to grow?

* _Extensibility_: Will people want to extend the behavior themselves?

* _Maintenance_: What is the lifecycle of this software? How long will it be
  around? Will decisions we make regarding the previous three bullet points
  "box us into any corners"? How can we avoid that?

## Interoperability

"integration" vs. "interoperability": what's the difference? People disagree:
[1](http://blog.capsuletech.com/integration-vs-interoperability-more-than-a-matter-of-semantics),
[2](https://www.securityworldmarket.com/int/News/Comment-of-the-Month/the-difference-between-integration-and-interoperability).

Here are my definitions:

* __Interoperability__: The ability of software components to consume one
  another's output as input. For example, Photoshop and ImageJ are
  _interoperable_ because they can each read and write TIFF files.

* __Integration__: The ability of software to utilize other software in its
  functioning. For example, KNIME can invoke ImageJ commands and scripts within
  its workflows. So we say ImageJ is _integrated_ into KNIME.

### How can we achieve interoperability?

Rule of thumb: start with command line functionality. With this approach, if
your software can run on the command line, with only files and stdin/stdout,
you achieve interoperability with a huge suite of tools structured the same
way.

* This is more powerful than a "GUI-centric" software.
  You can always build the GUI on top.
* Scales better: can run headless on a cluster. More important than ever.

Think about what the inputs and outputs are. Standards.

## Modularity

__Not modular enough__: A 10 GB software package that does everything. To gain
access to the functionality, you install the entire package. Each update is
very large, and therefore updates are pushed out much less frequently. MATLAB
is an example of software near this extreme.

__Too modular__: Thousands of tiny components that are updated independently.
To gain access to the desired functionality, dozens of components are
cherry-picked and combined. Functionality may jump between components.
Components might merge or split, adding to the complexity and confusion. Fiji,
with its hundreds of plugins and update sites, is closer to this extreme.

__The happy medium__: Divide functionality into pieces that go together. But
keep layers of isolation between unrelated areas of functionality, to ensure
proper separation of concerns.

### When should we split a component in half?

1. __Licensing.__ If different parts of the code need to be licensed
   differently. E.g., if one feature has adapted code licensed under the GPL,
   but the rest of the component's codebase does not need it.

2. __Dependencies.__ If different parts of the code have different
   dependencies. Especially if one small piece of the code makes uses of large
   dependencies that are unneeded elsewhere.

3. __Features and size.__ If the code contains multiple unrelated feature sets,
   or the codebase has become large enough that its purpose cannot be explained
   in a sentence or two. In that case, dividing into subcomponents avoids
   "bloat" and allows developers to cherry-pick the pieces they need.
   Another rule of thumb on the "per source file" level that I personally like
   is if it's more than 500 lines of code, split into two files.

### When to use a class, vs. functions?

More generally, the question is _when and how to group functions together_?
A class is really just a bag of functions and state.

Classes are useful when there is state. So: persistent information across functions.

* "A class should only do one thing."
* "A method or function should only do one thing."

Even without classes, _modules_ are very useful for grouping related
collections of functionality.

For naming: write out the calling code. See also Test Driven Development.

## Extensibility

__Not extensible enough__ - A closed source project with no public API,
offering an "all-in-one" black box solution for all needs. If the program falls
short, it cannot be fixed without requesting changes from the upstream
developer.

__Too extensible__ - Open source project where every implementation detail of
every plugin is exposed via public API functions. Too much API is overwhelming,
and creates [code paralysis](http://stackoverflow.com/a/3631338) that make
future improvements difficult without breaking existing extensions.

A good way to avoid the design becoming too extensible is to design with
[encapsulation](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming))
as a goal.

Think in terms of _interfaces_: the API contract. Do not expose anything else. 

### Dependency injection (DI) and application containers

* Code API to interfaces. : example: if there are multiple implementation for the same output based on hardware, the code can suggest preferenes in the meta-plugin file. a plugin using @ops.gauss* could use preference dialog to choose among different gauss filters gauss_CUDA, gauss_discrete, gauss_1D,etc.
* Create plugins or extensions implementing those interfaces.
* These implementations are made available in the runtime environment somehow.
* The framework takes care of providing the available best implementation,
  when downstream code wants an object conforming to that interface.
* See e.g. https://imagej.github.io/tutorials "Fundamentals of ImageJ" tutorial.

## Composition vs. inheritance

See https://stackoverflow.com/a/4913070



1) Encapsulation vs Interoperatibility : How to choose among keeping the version metadata in (Registry in windows) OR (config-metafiles) OR (User input) : User input is always better to identify version control. Another option is to work on the crumbs left there. eg: Java JRE leaves registry traces, https://stackoverflow.com/questions/5415485/how-to-remove-jre-entries-from-windows-registry that can be used to identify details. But always stick to readable plain text config files. eg: wiscscan chose *.config file instead of binaries.

2) Aplication containers: system architecture finds all the features assemble and give you a sack a tools
How to choose or index while "assembling a sack" of modules while startup: eg: Scijava annotation processes: source annotation protocol like the metafile json/org.scijava.plugin.plugin

3) API contract: limit to input/output. How to limit the function definitions within private? how does it scale to python with the underscores in naming. https://stackoverflow.com/questions/1301346/what-is-the-meaning-of-a-single-and-a-double-underscore-before-an-object-name

4) Dependency Injection + Modularity in practise (Inversion of Control) suggests a class should not configure its dependencies statically but should be configured from the outside.http://www.vogella.com/tutorials/DependencyInjection/article.html
eg: Google Guice



