When at [KIE][kie] team we started experimenting with [GraalVM]'s [native image][ni] 
compilation,  almost a couple of years ago, it started off as a small-scale exploration of a  brave new world; a side-project of a handful of developers, willing to try
something new and exciting in their spare time. Never would they have imagined it would 
become the  foundation for our next-generation, **cloud-native business automation platform**!

Today, [Kogito][kgt] has come a long way: we are excited to announce that we have
releasing **Kogito 1.0**! Through a combination of different technologies [Kogito][kgt] delivers a truly **low-code** platform for the design and implementation of knowledge-oriented REST services. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/2Ci_WcYtLrU" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

**Build-Time Optimization, Run-Time Joy!**
The [Kogito][kgt] platform embraces [Quarkus][qks]' AoT-first philosophy. The [Kogito extension][qex] generates the code to scaffold a complete REST service and, with developer mode, it supports live code reload not only for plain text, but even when you are using our newfangled graphic [modeling tools][kmd]!

![BPMN](https://raw.githubusercontent.com/kiegroup/kie-tooling-store/master/gifs/bpmn.gif)

Not ready for hopping on [Quarkus][qks]? Fear not: the Kogito Maven plug-in supports AoT code generation for [Spring Boot][spb], too.

**Standing on the Shoulders of Giants**. [Kogito][kgt] includes best-of-class support for the battle tested engines of
the [KIE][kie] platform: the [Drools][drl] rule language and decision platform, 
the [jBPM][jbpm] workflow and process automation engine and the [OptaPlanner][opt]
constraint satisfaction solver. 

Moreover we are infusing it with exciting new capabilities:
for instance, the [TrustyAI][tai] initiative for "trustable" Artificial Intelligence contributes services and tooling to get insights on machine-assisted decisions;
unique to Kogito is also the implementation of the [Serverless Workflow specification][sws], for cross-platform implementation of workflows in serverless environments. 

And even our well-known engines now support new exciting capabilities, such as a new type-safe format for rule definitions, a new PMML engine for machine learning workloads, and a new persistence layer for processes.


**Learn more**

- [KIE Live YouTube Channel][kielive]
- [Kogito Website][kgt]
- [KIE Zulip Chat][zlp]
- [Kogito Mailing List][kml]

[graalvm]: https://www.graalvm.org/
[ni]:  https://www.graalvm.org/reference-manual/native-image/
[kgt]: https://kogito.kie.org
[qex]: https://code.quarkus.io/
[qks]: https://quarkus.io
[kmd]: https://marketplace.visualstudio.com/items?itemName=kie-group.vscode-extension-kogito-bundle
[kol]: https://kiegroup.github.io/kogito-online/#/
[spb]: https://spring.io/projects/spring-boot
[kie]: http://kie.org
[drl]: http://www.drools.org/
[jbpm]: http://www.jbpm.org/
[opt]: http://www.optaplanner.org/
[tai]: https://blog.kie.org/2020/06/trusty-ai-introduction.html
[sws]: https://serverlessworkflow.io/
[kielive]: https://www.youtube.com/playlist?list=PLo3ZScdD9hW4S94iT3ZgOWm8asSHuMDYn
[zlp]: https://kie.zulipchat.com
[kml]: https://groups.google.com/forum/#!msgid/kogito-development/