---
layout: post
title:  "Java ClassLoader"
date:   2020-02-23 12:02:10 +0800
categories: jekyll update
---

### What is ClassLoader
> Class loaders are responsible for loading Java classes during runtime dynamically to the JVM (Java Virtual Machine). 
> Java classes aren't loaded into memory all at once, but when required by an application. This is where class loaders come into the picture. They are responsible for loading classes into memory.

### Types of Built-in Class Loaders
- Bootstrap ClassLoader
    - This bootstrap class loader is part of the core JVM and is written in native code
    - Bootstrap class loader serves as a parent of all the other ClassLoader instances.
    - It's mainly responsible for loading JDK internal classes, typically rt.jar and other core libraries located in $JAVA_HOME/jre/lib directory.
- Extension ClassLoader
    - Extension class loader is a child of the bootstrap class loader and takes care of loading the extensions of the standard core Java classes 
    - Extension class loader loads from the JDK extensions directory, usually $JAVA_HOME/lib/ext directory or any other directory mentioned in the java.ext.dirs system property.
- App ClassLoader(System ClassLoader)
    - The system or application class loader, on the other hand, takes care of loading all the application level classes into the JVM
    - It's a child of Extensions classloader.
    - It loads files found in the classpath environment variable, -classpath or -cp command line option

```java
public void printClassLoaders() throws ClassNotFoundException {
 
    System.out.println("Classloader of this class:"
        + PrintClassLoader.class.getClassLoader());
 
    System.out.println("Classloader of Logging:"
        + Logging.class.getClassLoader());
 
    System.out.println("Classloader of ArrayList:"
        + ArrayList.class.getClassLoader());
}
```
output:
```
Class loader of this class:sun.misc.Launcher$AppClassLoader@18b4aac2
Class loader of Logging:sun.misc.Launcher$ExtClassLoader@3caeaf62
Class loader of ArrayList:null
```

### How Do Class Loaders Work?
```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }
            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);
                // this is the defining class loader; record the stats
                PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```
- Delegation Model

    Class loaders follow the delegation model where on request to find a class or resource, a ClassLoader instance will delegate the search of the class or resource to the parent class loader.
- Unique Classes

    As a consequence of the delegation model, it's easy to ensure unique classes as we always try to delegate upwards.
- Visibility

    Children class loaders are visible to classes loaded by its parent class loaders.

### Custom ClassLoader
> Subclasses of ClassLoader are encouraged to override findClass(), rather than loadClass().

```java
public class CustomClassLoader extends ClassLoader {
 
    @Override
    public Class findClass(String name) throws ClassNotFoundException {
        byte[] b = loadClassFromFile(name);
        return defineClass(name, b, 0, b.length);
    }
 
    private byte[] loadClassFromFile(String fileName)  {
        InputStream inputStream = getClass().getClassLoader().getResourceAsStream(
                fileName.replace('.', File.separatorChar) + ".class");
        byte[] buffer;
        ByteArrayOutputStream byteStream = new ByteArrayOutputStream();
        int nextValue = 0;
        try {
            while ( (nextValue = inputStream.read()) != -1 ) {
                byteStream.write(nextValue);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        buffer = byteStream.toByteArray();
        return buffer;
    }
}
```

### References
[Class Loaders in Java](https://www.baeldung.com/java-classloaders)