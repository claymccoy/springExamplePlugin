![CI](https://github.com/spinnaker-plugin-examples/springExamplePlugin/workflows/CI/badge.svg)
![Latest Kork](https://github.com/spinnaker-plugin-examples/springExamplePlugin/workflows/Latest%20Kork/badge.svg?branch=master)
![Latest Orca](https://github.com/spinnaker-plugin-examples/springExamplePlugin/workflows/Latest%20Orca/badge.svg?branch=master)

This plugin exercises the Spring capabilities of Spinnaker plugins. There are no PF4J extension points or dependencies. Instead everything is Spring components.

It tests the following cases and all of it works without having to use Kork plugin's "unsafe" mode:
* new plugin beans are wired together
* app beans are autowired into plugin beans
* plugin beans are autowired into the app beans and replace app beans if primary (this allows modifying existing spinnaker behavior)
* properties are recognized
* controllers add new endpoints
* a service created using a factory bean for init logic based on other beans or properties
* new dependencies that are not in Spinnaker can be used in your plugin beans

Some things that don't work:
* the Configuration annotation won't be recognized in a plugin bean so you can't create beans that way
* class path scanning your plugin
* the Conditional annotation won't be recognized in a plugin bean
* Comnponent and Primary annotations aren't recognized, but that functionality still exists

Your plugin needs to extend PrivilegedSpringPlugin and implement registerBeanDefinitions(BeanDefinitionRegistry) to explicitly create and register BeanDefinitions. There are helper methods to make this easier and they default to singleton scope with constructor autowiring.  See SpringExamplePlugin for an example.

<h2>Usage</h2>

1) Run `./gradlew releaseBundle`
2) Put the `/build/distributions/<project>-<version>.zip` into the [configured plugins location for your service](https://pf4j.org/doc/packaging.html).
3) Configure the Spinnaker service. Put the following in the service yml to enable the plugin and configure the extension.
```
spinnaker:
  extensibility:
    plugins:
      Armory.SpringExamplePlugin:
        enabled: true

newproperties:
  test: 'test1'
```

Or use the [examplePluginRepository](https://github.com/spinnaker-plugin-examples/examplePluginRepository) to avoid copying the plugin `.zip` artifact.

To debug the plugin inside a Spinnaker service (like Orca) using IntelliJ Idea follow these steps:

1) Run `./gradlew releaseBundle` in the plugin project.
2) Copy the generated `.plugin-ref` file under `build` in the plugin project submodule for the service to the `plugins` directory under root in the Spinnaker service that will use the plugin .
3) Link the plugin project to the service project in IntelliJ (from the service project use the `+` button in the Gradle tab and select the plugin build.gradle).
4) Configure the Spinnaker service the same way specified above.
5) Create a new IntelliJ run configuration for the service that has the VM option `-Dpf4j.mode=development` and does a `Build Project` before launch.
6) Debug away...
