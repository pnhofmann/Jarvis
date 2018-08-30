# Plugins

## Basic style

There are two equivalent ways to declare Plugins

### Decorator-style
Shoud be used for **small** plugins.

```
from plugin import plugin

@plugin()
def hello_world(jarvis, s):
    """Prints \"hello world!\""""
    jarvis.say("Hello World!")
```

### Class-style
Should be used for more **complex** plugins.
```
from plugin import Plugin


class HelloWorld(Plugin):
    """
    Description of Hello world
    """
    def require(self):
        pass

    def complete(self):
        pass

    def alias(self):
        pass

    def run(self, jarvis, s):
        jarvis.say("Hello world!")
```

Ignore everything except "run" for now.


## Naming

* Class/Method Name = command you enter in Jarvis prompt
* Case-insenstive - Jarvis commands are always lower case
* Do not use "_"
* __ means "comment" - everything beyond is ignored

So these plugins are equivalent:

* HelloWorld
* helloworld
* HelloWorld__LINUX_ONLY

But **NOT**:

* hello_world

That means something totally [different](PLUGINS.md#two-word-commands).


## Run-Parameter

* 1: jarvis -> instance of CmdInterpreter.JarvisAPI. You'll probably most need say(text, color=None) to print/say stuff.
* 2: s -> string; what the user entered


## Features

### Alias

Example:

Class-style:
```
    def alias(self):
       yield "Hello"
       yield "Hi"
```
Decorator-style:
```
from plugin import plugin, alias


@plugin()
@alias("Hello", "Hi")
def helloWorld(jarvis, s):
   (...)
```

This would alias "HelloWorld" to "Hello" and "Hi".

An Alias behaves just like copy-pasting the whole plugin.

### Require

Not all plugins are compabible with every system. To specify compbability constraints, use the require-feature.

Plugins, which are non compabible are ignored and will not be displayed.

Current available requirements:

* Network: (True, False)
* Plattform: (plugin.LINUX, plugin.MACOS)
* Python: (plugin.PYTHON2, plugin.PYTHON3)
* Native: (any string)

Native means any native Binary in $PATH must exist. If multiple natives are specified (unlike other requirements) ALL must be fullfilled.

Example: Only support Python 3 with Linux; Require firefox and curl with active network connection.

Class-style:
```
from plugin import Plugin, PYTHON3, LINUX

    def require(self):
        yield("plattform", LINUX)
        yield("python", PYTHON3)
        yield("network", True)
        yield("native", ["firefox", "curl"])
```

Decorator-style:
```
from plugin import plugin, PYTHON3, LINUX


@plugin(plattform=LINUX, python=PYTHON3, network=True, native=["firefox", "curl"])
def helloWorld(jarvis, s):
    (...)
```

Special behaviour:

* If a plugin is incompabible because of missing native binary (and because of natives only), an notification shall be printed.
* If network=True is required, you won't need to try-catch requests.ConnectionError - this error is auto-cached with connection-error


### Completion

Use Completion feature (complete with TAB).

Class-style:
```
from plugin import Plugin

    def complete(self):
        yield "complete0"
        yield "complete1"
```

Decorator-style:
```
from plugin import plugin, complete


@plugin()
@complete("complete0", "complete1")
def helloWorld(jarvis, s):
    (...)
```


### Two word commands

Take "check ram" and "check weather". It is possible to implement these in two different plugins:

```
@plugin()
check_ram(jarvis, s):
    (...)


@plugin()
check_weather(jarvis, s):
    (...)
```

One '_' in class- or method-names split first word from second.

Two word commands even work with alias() (but use Space instead of '_'):

```
from plugin import Plugin


class HelloWorld(Plugin):
    """
    Description of Hello world
    """
    def require(self):
        pass

    def complete(self):
        pass

    def alias(self):
        yield("Hello world")

    def run(self, jarvis, s):
        jarvis.say("Hello world!")
```

Will make "Hello world" available.

Note that this only works for two-word commands - there is currenlty nothing like "three world commands".
