# Scaffolding Code in Kogito with Quarkus

The Kogito code generation process is "staged"; each stage is responsible for a different business asset (e.g. Processes, Rules, etc.). [Read more in this blog post][blog]. In the following:  

1. I am detailing the type of code that is generated in Kogito (`kogito-codegen`), then 
2. I will describe how this code generation is run inside the Kogito extension, and 
3. what are the limitations in the current extension execution flow to allow for the form of scaffolding that we would like to realize; then 
4. I will propose how we could bring this feature to Quarkus as a generic solution for continuous scaffolding

The intention is possibly to come up with a co-design and possibly share some of the implementation tasks, if necessary.

## Categorization of Generated Code

Each stage generates code that may fall in three different categories.

### (Inferred) Data Model

A generated data object (a POJO) that is inferred from a business asset. 
For instance, a Business Process defines process variables
which are implicitly turned into a POJO whose fields are those variables. 

e.g. variable `name` of type `String` in process `UserRegistration` may yield the class

```java
class UserRegistrationModel {
  String name; // get, set, constructors ...
}
```

Similarly for  DMN context, etc. 

**Customization.** These classes may be customized or user-provided in the future.

### Typed Knowledge-Asset API

Generated classes that wrap the internals  of the engines with a type-safe API, that is tailored over the Data Model;

e.g. `Process<UserRegistrationModel>`, `RuleUnit<MyUnitData>`

For instance:

- Rules generate source code for the RuleUnit implementation **and** the 
  "executable model" of these rules
- Processes generate source code for their Process implementation using
  their specific "executable model" description
- ...

The "executable model" in both cases is a representation through code
of these assets. It is an *internal implementation detail* and it is not
meant to be hand-written by end-users.


**Customization.**  These classes may NOT be customized or user-provided, as they are implementation-defined.

### REST Resources

Generated classes that wrap the typed knowledge-asset API,
and expose actions through REST endpoints; e.g. (pseudo-code)

```java
@Path("/user-registration")
class UserRegistrationResource { 
  @Inject 
  Process<UserRegistrationModel> p; /* ... endpoints ... */
  @POST
  public create(UserRegistrationModel m) {
    p.createInstance(m).start();
  }
}
```

**Customization.**  These classes may be customized or user-provided in the future.

## Why Generating Source Code

A common question is why we decided to generate **source code** instead of bytecode. There are several answers.

### Typed APIs and Executable Model

"Typed Knowledge-Asset APIs" expose the internal details of our engine through a higher-level API; as part of this code-generation process,
we also generate what we call the "executable model" of our engines. 

The "executable model" is a representation through code of business assets. For instance, for processes, it is a description in code of a workflow (basically, a directed graph); for rules, it is a description in code of these rules. In both cases, however, the executable model is intended as a *lowered* representation of the original source files, where some checks may have been already occurred (e.g. validation).

At the same time, this code is still "quite high-level", because, internally we rely on a few checks from the `javac` compiler to make sure that the code we generate composes correctly. If we decided to generate bytecode, we would end up redoing a lot of such work in our own code, to the point of almost duplicating the `javac` work. 

### Data Models and REST Resources

The value proposition for Kogito is to get a working, self-contained REST service out of business assets without extra effort. 

However, we always intended to allow users to customize the code that we generate, *for data models and REST resources*. In this case, generating _readable_ source code is valuable because it doubles as _code scaffolding_. 

In other words: 

- we can generate source code and *compile it on-the-fly* to generate a self-contained REST service. In this case the code is an implementation detail that ends into the build output directory (usually `target/`)

- we can let the user "opt-out" of this process and let the user "own" that generated code for a given asset. In this case, the code would end in a source directory (e.g. `src/main/java`) so that users can customize it at will. "Opting out" means that we stop generating code for that asset, and the user becomes responsible for maintaining those bits. 

E.g. `class UserRegistrationModel` and `class UserRegistrationResource`

## Compilation Process in the Quarkus Extension

The Kogito code-generation process, makes the framework akin to a domain-specific language compiler. Earlier protoypes of the Quarkus extension, mimicked the behavior of other language-compiling extensions, such as Scala's and Kotlin. These extensions require users to add a third-party plug-in with support for that language. The extension in this case usually only provides `CompilationProviders` specific to that language.

However, we decided that requiring users to both add a Maven plug-in and a Quarkus-extension would have been cumbersome, because, although business automation assets _are_, in a way, domain-specific languages, users _do not_ perceive them as "real" programming languages. As a result, we decided that the Quarkus extension would have worked seamlessly without requiring extra dependencies or plug-ins.

In order to do so, we are processing the resources in two separate ways, depending whether we're running during Quarkus' "boot-up" time (augmentation phase) or developer mode (hot-reload). 

### Augmentation

In this phase, the `KogitoAssetsProcess` is the main entry point of the Quarkus extension; we gather the business assets (usually they are in `src/main/resources`), generate code and compile it on-the-fly.

This is currently done using an in-memory compiler, so that generated class files can be then returned to the Quarkus extension using a `GeneratedBeanBuildItem`. An alternative would be to just delegate Java code compilation to the "regular" Java compiler: we are not doing this, because, originally, then Quarkus would not pick up the generated classes. On the other hand, this is what we are doing for hot reload.

Caveat: because this code runs at `package` time, it **must** be treated as an implementation detail, as user-facing code, cannot refer to it directly. This has a direct impact on scaffolding (discussed later).

### Hot Reload

In this case, `KogitoAssetsProcess` exits earlier, because only `CompilationProvider`s get notified of diffed changes. So, we registered a `CompilationProvider` which delegates to (inherits from) a `JavaCompilationProvider`. We dump the generated code to disk, then let it compile to the delegate `javac`. In this way, we are sure that only changes are recompiled.

### Shared Code and Relation to Scaffolding

Because both phases are actually executing code that lives in `kogito-codegen`, internally, the code generation process is pretty much the same, even though it is invoked by different actors at different stages. 

As mentioned above, the code that we generate is source code; the intention is to reuse the same code-generation process to promote such code to "scaffolding". However, in this case, we should generate such code in a user source directory and/or run the code generation procedure _before_ the Maven `package` phase. However, because right now the Quarkus Maven plug-in is always tied to `package` phase, user-facing code cannot refer to this code directly. 

For instance, suppose you want to write a test referring to a `Process<FooModel>`. `FooModel` is generated code, but such code is generated in the `package` phase: the result is that the test fails to compile! Today, we are resorting to testing the REST API instead. But, if users decided to opt-out of REST endpoint generation, then, those tests would become much harder to write.

### Work-Around

As a work-around, we may ask our users to install our Maven plug-in, which essentially duplicates the Quarkus extension, but it allows to invoke Mojos directly; thereby allowing for a finer degree of control of when/where executing such code generation tasks.

## Continuous Scaffolding

As already describe above, the code-generation procedure is already independent from the Quarkus extension API; thus, we can easily invoke it in the augmentation phase or dev-mode. 

In a [continuous scaffolding](https://github.com/quarkusio/quarkus/issues/6025) scenario, users may be allowed to mark a resource for processing _before_ the augmentation phase. This could be automatic and tightly-connected to hot-reload mode, or explicit.

- **Automatic.** a file type is "claimed" by a Quarkus extension. If one such file type is present in the project, the "scaffolding" portion of the extension runs *before compile*. After Quarkus boot-up, code may be regenerated as part of hot-reload. In both cases, `javac` is invoked to compile the resource. In the first case, it may be delegated to the Maven compiler plug-in; in the second case, it would responsibility of the Compilation Provider.

- **Explicit.** A mechanism is chosen to claim a specific file or collection of files. E.g., a directory, a special naming convention, a system or config property, etc. The "scaffolding" portion of the extension runs *before compile* and/or for a given `command` (e.g. `-Dquarkus.kogito.generate=...`). No further code-generation is performed either after Quarkus boot-up or subsequent runs of the plug-in. 

Either way, I could see how both modes could be realized with the same code-base; i.e., explicit is just a restricted version of the automatic scheme / the automatic scheme reuses the "explicit" case, using instead the `CompilationProvider` trick to get diffed list of files (described above). If this mechanism was supported more generally in Quarkus, the Kogito extension may even entirely drop the "augmentation" part.



[blog]: https://evacchi.github.io/compilers/kogito/2020/04/23/kogito-codegen-design.html
