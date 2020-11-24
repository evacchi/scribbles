For the last few month, here at [KIE][kie] team we've been hard at work; and that is one of the reasons I haven't written something new for a while. Today I am proud to announce that our **cloud-native business automation platform** is hitting a major milestone. **Today we release [Kogito 1.0][kgt]**! 

[Kogito][kgt] includes best-of-class support for the battle tested engines of
the [KIE][kie] platform: 

- the [Drools][drl] rule language and decision platform, 
- the [jBPM][jbpm] workflow and process automation engine 
- the [OptaPlanner][opt] constraint satisfaction solver

and it brings along new capabilities 
- our fresh new unified [on-line BPMN and DMN editors](https://kiegroup.github.io/kogito-online/#/) and [VSCode-based extension](https://marketplace.visualstudio.com/items?itemName=kie-group.vscode-extension-kogito-bundle)
- noSQL persistence through the Infinispan and the MongoDB addons
- GraphQL status queries
- microservice-based data indexing and timer management
- completely revisited UIs for task and process state 
- CloudEvent for event handling
- the new vendor-neutral [Serverless Workflow Specification](sws)
- business-relevant insights on machine-assisted decisions through the contributions of the [TrustyAI](tai) initiative
- automated deployment through the [Kogito Operator][kop] and the [`kogito` CLI][kli]


[![Kogito Presentation Video](https://img.youtube.com/vi/2Ci_WcYtLrU/0.jpg)](https://www.youtube.com/watch?v=2Ci_WcYtLrU)


I believe there is a lot to be proud of, but I want to talk more about another of the things that make Kogito special, and that is the **heavy reliance on code-generation**. 

In Kogito code-generation has a double purpose: 
1. we generate code ahead-of-time to avoid run-time reflection; 
2. we automatically generate domain-specific services from user-provided knowledge assets.

Together, [Kogito][kgt] delivers a truly **low-code** platform for the design and implementation of knowledge-oriented REST services. 

## Ahead-of-Time Code-Generation

In [Kogito][kgt], we **load, parse, analyze** your knowledge assets such as rules, decisions or workflow definitions during your **build-time**. This way, your application starts faster and it consumes less memory, and, at run-time, it won't do more than what's necessary.

![flow1](imgs/flow2.png)

Compare this to a **more traditional pipeline**, where instead the all the stages of processing of a knowledge asset would occur at run-time:

![flow1](imgs/flow1.png)


The **Cloud**, albeit allegedly being just "someone else's computer", today is a reality. More and more businesses are using cloud platforms to deploy and run their services. And consequently, because they are paying for the resources they use, they are caring more and more about them.

![I am your density](imgs/density.png)

This is why *application density* is becoming increasingly more important: we want to fit more application instances on the same host, because we want to keep costs low. If your application has a huge memory footprint and high CPU requirements, it will cost more.

While we do support [Spring Boot][spb] (because, hey, we realize you can't really ignore a 800-pound gorilla) we chose [Quarkus][qks] as our primary target runtime, because through its extension system, it lets us truly **embrace ahead-of-time code generation**.

Whichever you choose, be it Spring, or Quarkus, Kogito will move as much processing as possible at build time. But if you want to get the most out of it, we invite you to give Quarkus a try: through its simplified support to native image generation, it lets us go even further, producing the **tiniest native executables**. So tiny and cute, they are the envy of a [gopher](https://blog.golang.org/gopher).

We cut the fat, but you won't lose flavor. Plus, with Quarkus, you'll get **live code reload** for free.


## Automated Generation of Services

Although build-time processing is a characterizing trait of Kogito, code-generation is also key to another aspect. We automatically generate a service starting from the knowledge assets that users provide.

You write rules, a DMN decision, a BPMN process or a serverless workflow: in all these cases, in order for these resources to be consumed, you need an API to be provided. In the past, you had full access to the power of our engines, through  a [command-based REST API for remote execution](#insert-url-here) or through their Java programmatic API, when embedding them in a larger application.

While programmatic interaction will always be possible (and we are constantly improving it in Kogito to make it better), in Kogito we aim for **low-code**. You drop your business assets in a folder, start the build process, and you get a working service running.

![Kogito Codegen](imgs/kogito-codegen.gif)


To the point where you can get from a knowledge asset to a fully-working service in a matter of one click or one command. In this animation you can see the `kogito` cli in action: the operator picks up the knowledge assets, builds a container and deploys it to openshift with just 3 commands!


```







        Placeholder: Operator







```


For local development, the [Kogito Quarkus extension][qex] in developer mode extends Quarkus' native live code reloading capabilities going further from reloading of usual plain-text source code (such as Java, Kotlin or Scala) to adding support to hot reload of graphical models supported by our [modeling tools](https://marketplace.visualstudio.com/items?itemName=kie-group.vscode-extension-kogito-bundle). In this animation, for instance you can see hot-reload of a DMN decision table

![BPMN](imgs/hot-reload.gif)

In the future we plan to support customization of these automatically-generated services, with a feature we call **scaffolding**. With scaffolding...

## Conclusions

Kogito 1.0 brings a lot of features..., we are excited to...


- [KIE Live YouTube Channel][kielive]
- [Kogito Website][kgt]
- [KIE Zulip Chat][zlp]
- [Kogito Mailing List][kml]


[kgt]: https://kogito.kie.org
[kie]: http://kie.org
[drl]: http://www.drools.org/
[jbpm]: http://www.jbpm.org/
[opt]: http://www.optaplanner.org/
[sws]: https://serverlessworkflow.io/
[tai]: https://blog.kie.org/2020/06/trusty-ai-introduction.html
[kop]: https://operatorhub.io/operator/kogito-operator 
[kli]: https://github.com/kiegroup/kogito-cloud-operator/blob/master/README.md

[qks]: https://quarkus.io
[spb]: https://spring.io/projects/spring-boot

[kielive]: https://www.youtube.com/playlist?list=PLo3ZScdD9hW4S94iT3ZgOWm8asSHuMDYn
[zlp]: https://kie.zulipchat.com
[kml]: https://groups.google.com/forum/#!msgid/kogito-development/
