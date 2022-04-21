# Background

Currently, there is no consistent way for SBT plug-ins to be loaded in enterprise environments.

The background is that SBT plug-ins originally were published in an Ivy (non-Maven) repository, and so the naming format convention lead to artifacts being published to Maven Central using workarounds, which require further workarounds in more restricted environments that depend on a strict Maven convention.

All Scala libraries follow the standard POM consistency convention, however SBT plugins do not. For example:

```
https://repo1.maven.org/maven2/ch/epfl/scala/sbt-web-scalajs-bundler_2.12_1.0/0.20.0/sbt-web-scalajs-bundler-0.20.0.pom
```

To be downloadable as a normal Maven dependency, the final filename would need to include the full artifact ID:

```
https://repo1.maven.org/maven2/ch/epfl/scala/sbt-web-scalajs-bundler_2.12_1.0/0.20.0/sbt-web-scalajs-bundler_2.12_1.0-0.20.0.pom
```

Otherwise, this plugin is inaccessible in a corporate environment (think big banks).

# Impact

The impact of this is that Scala cannot be fully adopted in these organizations. An organization with 20,000 Java developers has 200 times 100 smaller companies of potential Scala users, 
but if they cannot use Scala to its fullest extent, this is a huge loss of opportunity - I might even claim that this is a major reason why Scala has been shunned.

To go outside the norms in corporate environments, due to them being responsible for vastly more sensitive information than many small companies, the security vetting process is strict and demanding: it can take a determined individual half a year to get past this barrier.
However there are not many of these, as developers rather get on with coding than making a case to security experts why this should be allowed, especially for a specialized non-major language.

By fixing this, the accessibility of Scala could increase massively across many organizations, as configuring SBT to work at any major corporation would be very easy, if it simply follows standard Maven set-up, simply because Maven is universally supported.

# Potential solutions

Given that internal solutions are not scalable (only within 1 organization), a more uniform solution needs to be applied across the board.

Eugene Yokota has described an interesting, yet somewhat custom process, to publish SBT plugins with consistency: https://eed3si9n.com/pom-consistency-for-sbt-plugins/

He also has done this for the sbt-assembly plugin (this has been verified to work in a highly rigid set-up):

Consistent publishing:

```
https://repo1.maven.org/maven2/com/eed3si9n/sbt-assembly_sbt1_2.12/1.2.0/sbt-assembly_sbt1_2.12-1.2.0.pom
```

Inconsistent publishing:
```
https://repo1.maven.org/maven2/com/eed3si9n/sbt-assembly_2.12_1.0/1.2.0/sbt-assembly-1.2.0.pom
```

Is there a way to publish both of these at the same time, so that plugin authors could simply publish them out?

There are 2 questions to bear in mind:
- How do you consume these artifacts in a consistent fashion? So that developers can simply use `addSbtPlugin(...)` whether using the consistent format or not
- How do you handle plugin dependencies? eg `sbt-web-scalajs-bundler` has a dependency on another plugin jar which is actually an invalid dependency, only valid from SBT's perspective through `additional` attributes (a hack from a Maven point of view).

Following solutions could be applied here:
1. We could automate a publishing matrix to publish in both consistent and inconsistent formats, for library authors.
2. We could republish plugins in a consistent format, but under a clean groupId, and potentially rewrite `addSbtPlugin()` dependencies at resolution time into republished ones.
3. Users could be given the option to configure in their SBT to say whether they want to follow the consistent POM format by default -- meaning what Eugene has in mind with SBT 2, but done far sooner.
   This would enable a quick diagnosis of what is missing in terms of plugins.

These options could all be combined together, so that plugins could be accessible in a consistent way for people in all sorts of environments, while keeping in mind the future compatibility very eagerly.

## Another potentially interesting solution

Given that you can publish inconsistent POMs, is it possible to attach multiple POMs to an artifact? Ie so that you have the following:

```
https://repo1.maven.org/maven2/com/eed3si9n/sbt-assembly_2.12_1.0/1.2.0/sbt-assembly-1.2.0.pom
https://repo1.maven.org/maven2/com/eed3si9n/sbt-assembly_2.12_1.0/1.2.0/sbt-assembly_2.12_1.0-1.2.0.pom
```

This way, you could have a single publish task that publishes both a standard version and a non-standard version. In this way, the effort required on the library authors could be much less, to the degree of possibly just adding an SBT plugin or upgrading to a newer SBT version, if this plugin is made into a default one.

This needs to be validated, but I suspect it will actually work. Effectively, it would work for older SBT users, but if you are a newer SBT user, you could specify an option `downloadConsistent := true`, and SBT would only look for those that are consistent, thus working in a corporate proxied environment with only 1 main change. It might be possible as well to have fallbacks in resolution but this needs to be validated.

This is the first experiment worth trying. If it works, then getting plug-in authors to publish in this approach would solve the problem rather quickly.

Eugene's proposal is good but we should try to unify the syntax of adding SBT plugins, so that users do not get confused over this whole consistency thing by seeing various suffixes in artifacts.

## Republishing plugins

I did an experiment, and it worked to republish plugins into Maven Central. We could build on this and expand into the idea described prior: we can try to republish quite a few plug-ins, without modifying original JARs; just repackaging with new POMs. This would require a change into a `libraryDependencies += ` syntax, although it might be possible to specify a new `addSbtPlugin()` implementation as well, specific to Maven-consistent plugins. Republishing is also frustrating because we would need to repackage and keep up with developments in each of the SBT plugins, eg IntelliJ SBT plugins.

This is a good tactical fix, but it is definitely not scalable.

# Preferred approach

We should head to publishing both styles as soon as possible, in order to retain backwards and forwards compatibility. Perhaps after 1 year, all users would be migrated to the new style, and a switch could be made to use the defaults.

## Can we wait until SBT 2.0?

No. There are only disadvantages to waiting. Time lost is time lost to other languages that have a better developer tooling UX, even if they are not actually better as languages.
