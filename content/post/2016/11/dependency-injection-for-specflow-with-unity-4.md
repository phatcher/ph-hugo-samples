---
title: "Dependency Injection For SpecFlow With Unity 4"
date: 2016-11-11 12:56:00Z
draft: false
aliases:
- /post/dependency-injection-for-specflow-with-unity-4/
categories:
- Testing
- Integration
---
One of my current projects is using SpecFlow for testing some complex security logic; there are lots of scenarios so the BDD style of testing is well suited to the problem. I wanted to use the same style of testing for some integration testing where the client is talking to my API but hit a problem of how/where to construct the container as even the client library uses dependency injection to construct itself.

A search quickly found [SpecFlow.Autofac](https://www.nuget.org/packages/SpecFlow.Autofac) by Gaspar Nagy, which provides a plugin to allow AutoFac to be used as a container by SpecFlow. We are using Unity in this project so I used the code at at https://github.com/gasparnagy/SpecFlow.Autofac to create the equivalent project for Unity 4 with the inspiring name of [SpecFlow.Unity](https://www.nuget.org/packages/SpecFlow.Unity) :smile:. I’m not going to explain dependency injection yet again, just a short note on how to use the plugin.

## Installing the package
First add the package from NuGet into your SpecFlow project
```
PM> Install-Package SpecFlow.Unity
```

## Configuring the container
Unlike AutoFac, Unity doesn’t have builder methods without going to Unity extensions, and I didn’t want to impose a particular style of construction on consumers of the plugin, so I presume that you already have a way of constructing an appropriate container. 

What we do need to do however is make this container available to SpecFlow and also add some additional registrations so that the step definitions are produced by Unity and have their dependencies resolved.  To do this, you need to create a static method in your SpecFlow project and annotate it with a ScenarioDependencies attribute – the plugin has code that will automatically locate this and perform the appropriate configuration in SpecFlow. 

A typical method would look something like this...

```
public static class TestDependencies
{ 
     [ScenarioDependencies] 
     public static IUnityContainer CreateContainer() 
     { 
         // create container with the runtime dependencies 
         var container = Dependencies.CreateContainer(); 
  
         // TODO: add customizations, stubs required for testing 
  
         // Registers the build steps, this gives us dependency resolution using the container. 
         // NB If you need named parameters into the steps you should override specific registrations 
         container.RegisterTypes(typeof(TestDependencies).Assembly.GetTypes().Where(t => Attribute.IsDefined(t, typeof(BindingAttribute))), 
                                 WithMappings.FromMatchingInterface, 
                                 WithName.Default, 
                                 WithLifetime.ContainerControlled); 
         return container; 
     }
}
```

To register the steps I’ve provided a little extension method in the plugin that does wraps this so instead you can do

```
container.RegisterStepDefinitions<StepClass>();
```

One other point is that if you need named dependencies injected into your steps you will have to re-register the step and give the dependencies explicitly. Also you must also have at least one step definition in the same assembly as the ScenarioDependencies method for the it to be found. 

## SpecFlow Config
The final step is to adjust your SpecFlow config file so that it knows you are using the plugin

```
<specFlow> 
    <!-- For additional details on SpecFlow configuration options see http://go.specflow.org/doc-config –-> 
    <unitTestProvider name="NUnit" /> 
    <runtime stopAtFirstError="false" missingOrPendingStepsOutcome="Inconclusive" /> 
    <stepAssemblies> 
        <stepAssembly assembly="StepAssembly" /> 
    </stepAssemblies> 
    <plugins> 
        <add name="SpecFlow.Unity" type="Runtime" /> 
    </plugins> 
</specFlow>
```

There’s a full sample project at https://github.com/phatcher/SpecFlow.Unity/tree/master/sample/MyCalculator

Now you have a configured container and steps which are using dependency injection. Finally let me know how you get on at https://github.com/phatcher/SpecFlow.Unity and if there’s any demand for a version for Unity 3.5. 
