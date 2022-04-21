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
- How do you handle plugin dependencies? eg `sbt-web-scalajs-bundler` has a dependency on another plugin jar which is actually an invalid dependency.

Following solutions could be applied here:
1. We could automate a publishing matrix to publish in both consistent and inconsistent formats, for library authors.
2. We could republish plugins in a consistent format, but under a clean groupId, and potentially rewrite `addSbtPlugin()` dependencies at resolution time into republished ones.
3. Users could be given the option to configure in their SBT to say whether they want to follow the consistent POM format by default -- meaning what Eugene has in mind with SBT 2, but done far sooner.
   This would enable a quick diagnosis of what is missing in terms of plugins.

These options could all be combined together, so that plugins could be accessible in a consistent way for people in all sorts of environments, while keeping in mind the future compatibility very eagerly.
