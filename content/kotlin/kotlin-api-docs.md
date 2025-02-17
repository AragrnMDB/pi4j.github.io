---
title: Kotlin API & DSL Documentation
description: "Pi4J Kotlin DSL & API Documentation"
weight: 47
---

{{% notice info %}}
Here you can find the provided APIs and the dead-simple DSLs of the Kotlin package.
{{% /notice %}}


## Components

### Context

To create a new Pi4J context, use the `pi4j` function. It creates a new _auto_ `Context` object and uses it as a
receiver
for your lambda:

```kotlin
pi4j {
    // You have access to a newly created auto context 
    describe()
}
```

You don't need to call `shutdown()`, it's called automatically at the end of the block.  
You can think of the `pi4j` block as your entire routine/ program.

#### Custom Context

You can build a custom context using the `buildContext` function:

```kotlin
val ctx = buildContext {
    +MockPlatform() // add a platform to our context
    +MockPwmProvider.newInstance() // add a provider to our context
    +File("tmp.txt") // add a file property
    +("key" to "value") // add a String property
} 
```

You can add other properties the same way like `InputStream`, `Reader`, and `Properties` instances.

#### Generics

You can also have access to `Context` properties using generics and type safe methods:

```kotlin
context.run {
    hasPlatform<T>() // equivalent to Context::hasPlatform(Type::class.java)
    hasProvider<T>() // equivalent to Context::hasProvider(Type::class.java)
    provider<T>() // equivalent to Context::provider(Type::class.java)
}
```

---

### I/O

You can easily create and configure I/O pins using this DSL

#### Digital I/O

From any Context object you can create I/O instances using:

```kotlin
digitalInput(address = 24) {
    name("Button")
    pull(PullResistance.PULL_DOWN)
    debounce(3000L)
}

digitalOutput(22) {
    name("LED Flasher")
    initial(DigitalState.LOW)
}
```

Using this DSL you get access to convenient functions like:

```kotlin
digitalInput(24).run {
    listen {
        // listens on state changes
        val currentState = it.state()
    }

    onLow {
        // fires when state changes to DigitalState.LOW
    }

    onHigh {
        // fires when state changes to DigitalState.High
    }

    if (isLow) {
        // you get isLow property 
    }

    if (isHigh) {
        // you get isHigh property 
    }
}
```

Also, when you need to specify a provider for the pin you're creating you can use these 2 safe providers:

```kotlin
digitalOutput(22) {
    mockProvider() // uses the mock provider
    piGpioProvider() // uses the pi_gpio provider
}
```

#### Analog I/O

There are common APIs between Analog I/O and Digital I/O like:

```kotlin
analogInput(address = 24) {
    name("Button")
    mockProvider() // uses the mock provider
    piGpioProvider() // uses the pi_gpio provider
}.run {
    listen {
        // listens on value changes
        val currentValue = it.value()
    }
}
```

However, Analog I/O get their own unique treats:

```kotlin
analogOutput(24).run {
    whenInRange(0..5) {
        // fires when value is in the supplied range
    }

    whenOutOfRange(0..5) {
        // fires when value is not in the supplied range
    }

    onMin(0..5) {
        // fires when value changes to the minimum of the range
    }

    onMax(0..5) {
        // fires when value changes to the maximum of the range
    }
}
```

#### PWM
PWM has its share of love as well: 

```kotlin
pwm(address = 24) {
    frequency(10_000)
    dutyCycle(40)
    mockProvider() // uses the mock provider
    piGpioProvider() // uses the pi_gpio provider
}
```

---

### Console

Using the `console` function, you can create a `Console` object and use it to print output to the console:

```kotlin
console {
    +"This will be printed as a new line"
    box("This will be printed inside a box")
    +"Another line because I really like it"
}
```

You also get other helper functions found in the Minimal Example like `printLoadedPlatforms`, `printDefaultPlatform`
, `printProviders`, and `printRegistry`.

---

### Platform
There are some convenient functions added to the `Platform` API

#### Generics
```kotlin
context.platform<MockPlatform>().run {
    hasProvider<T>() // equivalent to Platform::hasProvider(Type::class.java)
    provider<T>() // equivalent to Platform::provider(Type::class.java)
    createFrom<IO>(IOConfig<*>) // equivalent to Platform::create(IOConfig<*> ,IO::class.java)
    createFrom<IO>(String) // equivalent to Platform::create(String ,IO::class.java)
}
```
