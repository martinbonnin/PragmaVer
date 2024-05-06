
# ~~PragmaVer~~

> [!IMPORTANT]
> 2024-04: I changed my mind and use proper semver now. See [martinbonnin/library-guidelines](https://github.com/martinbonnin/library-guidelines)

### A pragmatic versioning scheme for artifacts consumed by Gradle builds

I recently had [an interesting discussion](https://kotlinlang.slack.com/archives/C8C4JTXR7/p1626082769022200) about semantic versioning for Maven artifacts. While there seem to be a consensus for versions in the form of `MAJOR.MINOR.PATCH`, things become more complex and interesting when pre-release are involved and multiple forms are found in the wild. For example:

* `3.0.0-alpha1`
* `3.0.0-alpha01`
* `3.0.0-alpha.1`

Which one is used depends on the projects. Ultimately, I chose `3.0.0-alpha01` because this is what other [popular libaries](https://maven.google.com/web/index.html?q=compose#androidx.compose.ui:ui) do and this will most likely be the least surprising choice for consumers. 

This page is an attempt at formalizing a versioning scheme for artifacts consumed with Gradle. It's unfortunately not compatible with SemVer but there's so much muscle memory around these that changing them seems counterproductive at the time of writing. Maybe one day...

I'll use this page as a reminder when I will have lost all context in 6 months from now. Also, I'm hoping others can find it useful or amend it. Feedback is very welcome!

## Proposal

From smaller to bigger (using [Gradle rules](https://docs.gradle.org/current/userguide/single_versions.html#version_ordering)):

* `3.0.0-dev01` 
* `3.0.0-alpha01`
* `3.0.0-beta01`
* `3.0.0-rc01`
* `3.0.0-SNAPSHOT`
* `3.0.0`

## Incompatibilities with SemVer

While the above works well with Gradle, it is incompatible with SemVer:
* Gradle has a [special rule](https://github.com/gradle/gradle/blob/5ec3f672ed600a86280be490395d70b7bc634862/subprojects/dependency-management/src/main/java/org/gradle/api/internal/artifacts/ivyservice/ivyresolve/strategy/StaticVersionComparator.java#L32) for `"dev"` where dev is always before anything else, even `"alpha"`
* Gradle also has a special rule for `"SNAPSHOT"` where the comparison is [case-insensitive](https://github.com/gradle/gradle/blob/5ec3f672ed600a86280be490395d70b7bc634862/subprojects/dependency-management/src/main/java/org/gradle/api/internal/artifacts/ivyservice/ivyresolve/strategy/StaticVersionComparator.java#L80). `"SNAPSHOT"` will come after `"alpha"` even though it shouldn't be in pure ASCII sort order.

All in all, [SemVer](https://semver.org/) would sort versions in the following order:

* `3.0.0-SNAPSHOT`
* `3.0.0-alpha01`
* `3.0.0-beta01`
* `3.0.0-dev01`
* `3.0.0-rc01`
* `3.0.0`

Note how:
* Semver doesn't have any special rule for `SNAPSHOT` or `dev`
* Uppercase `SNAPSHOT` will come before `alpha01`
* `dev` will come after `beta`

# Is this important?

Maybe not! It seems the incompatibilities are not important **as long as no library transitively depends on a pre-release version**. Which sounds like a relatively bad idea in the first place since pre-releases could have breaking API changes. So in most cases, I would expect the PragmaVer scheme to not break any build.

There might still be some impact for some tools. I'm thinking of tools like [dependabot](https://dependabot.com/),  [ben-manes/gradle-versions-plugin](https://github.com/ben-manes/gradle-versions-plugin) or [jmfayard/refreshVersions](https://github.com/jmfayard/refreshVersions). Depending what ordering rules they use, these tools might suggest a `-dev` version while an `-alpha` is already released.

To some extent, the Maven Central HTML listings at https://repo1.maven.org/maven2/org/jetbrains/kotlin/kotlin-stdlib/ are also somewhat affected because they always sort in ASCII order and don't know about numeric parts. These listings are a 3rd ordering on top of the Gradle order and the SemVer order.

## A brighter future 🦄?

In the future it would be nice to use a versioning scheme and ordering rules that works the same for all tools. I think the following would work with both Gradle and SemVer:


* `3.0.0-1` (developer preview only use a single numerical part so they come before everything else)
* `3.0.0-alpha.1` (no leading zero)
* `3.0.0-beta.1`
* `3.0.0-rc.1`
* `3.0.0-snapshot` (lowerspace so that it comes after the rest)
* `3.0.0`

That would not work in Maven Central HTML listings but maybe Sonatype can make it work! 🙏

## Discussion

**Q**: Why not start at `3.0.0-alpha00`?

Good question! Because no one else does I guess...

**Q**: What happens if you need 100 alphas or more?

Well, at this point you'll have to go beta!

## Resources

* Semver: https://semver.org/
* Npm rules: https://docs.npmjs.com/cli/v6/using-npm/semver
* Gradle rules: https://docs.gradle.org/current/userguide/single_versions.html#version_ordering


_PS_: Many Thanks to [Javier](https://twitter.com/JavierSegoviaCo), [LouisCAD](https://twitter.com/louis_cad?lang=en), [AndroidHamilton](https://twitter.com/AndroidHamilton), [Dariusz](https://twitter.com/darek_kuc) and [Björn](https://twitter.com/_Vampire0_) for the interesting discussion!
