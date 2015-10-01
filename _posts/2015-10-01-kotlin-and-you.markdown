---
layout: post
date: 2015-10-01
title: "Kotlin and You"
toolbar: "woods.jpg"
---

I hear people like Kotlin. I don't blame them, it's a nice language to play with. Imagine Java without the junk. Yeah. That. So, can we use it with Forge?

**Yes**, yes we can. It isn't even that complicated, I promise. It's just a few tricks in Gradle and we're all set to go.

For the purposes of this tutorial, I'm going to assume a few things:

- You have the JDK (7+) installed and working.
- You use IntelliJ IDEA (I'll make a note about this later if not)
- Your project is using Gradle (unlike certain projects I can mention...)

Okay, checklist over, let's go!

##1. Build Dependencies
In your buildscript block in `build.gradle`, you'll need a dependencies block to load the Gradle Kotlin plugin, like so:

{% highlight groovy %}
buildscript {
  ext {
    [...]
    kotlin_ver = "0.14.449"
  }
  dependencies {
    [...]
    classpath "org.jeybrains.kotlin.kotlin-gradle-plugin:$kotlin_ver"
  }
}
{% endhighlight %}

Now, you don't _have_ to use the `ext` variable, but it'll be useful later. I recommend keeping it. If not, substitute $kotlin_ver for whatever version.

##2. Plugins
Next up, we need to apply our Gradle plugins after `buildscript {}`

{% highlight groovy %}
apply plugin: 'kotlin'
apply plugin: 'forge'
{% endhighlight %}

##3. Minecraft
Go read the Forge docs and then come back. You'll need a fully populated `minecraft {}` block for what comes next.

##4. Configurations and Dependencies
Welcome back. Next, we have to specify our build configurations and dependencies, in our case this is Kotlin. Somewhere after the plugins:

{% highlight groovy %}
configurations {
    shade
    compile.extendsFrom shade
}

dependencies {
    shade "org.jetbrains.kotlin:kotlin-stdlib:${kotlin_ver}"
    shade "org.jetbrains.kotlin:kotlin-reflect:${kotlin_ver}"
}
{% endhighlight %}

Remember I said `kotlin_ver` would come in handy? Yeah. Makes updating easier down the road.

What we just did was create a new dependency configuration (extending the usual `compile` one) and declare that we need Kotlins jars (`stdlib` and `reflect`).

##5. Repackage
This is the important bit. **Don't** skip this, it may make some poor sods game crash when loaded with another Kotlin mod. Add this to your `minecraft {}` block:

{% highlight groovy %}
minecraft {
  [...]
  srgExtra "PK: kotlin your/package/here/kotlin"
}
{% endhighlight %}

Obviously replace `your/package/here` with, well, your package. This prevents clashes with other Kotlin installations on the classpath.

##6. Shade
This is the part that actually stores Kotlin in your mod jar. If you already have a `jar` task defined, just add this in. If not, make the task like so:

{% highlight groovy %}
jar {
    // Shading
    configurations.shade.each { dep ->
        from(project.zipTree(dep)){
            exclude 'META-INF', 'META-INF/**'
        }
    }
}
{% endhighlight %}

##7. Hackhackhack
Make a Kotlin class like you would a standard Java one, annotate with @Mod (specifying `modLanguage="kotlin"` just in case) and hack away.

_fin_