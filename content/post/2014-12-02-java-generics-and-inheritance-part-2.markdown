---
author: Darío García
categories:
- java
- generics
- oop
- reflection
comments: true
date: 2014-12-02T17:10:14Z
title: Java generics and inheritance (part 2)
url: /2014/12/02/java-generics-and-inheritance-part-2/
---

[On the first part of this article](/2014/12/02/java-generics-and-inheritance-part-1) I explained why java generics don't allow up-casting for generified types.
In this part we will see why, arrays don't have that restriction, and the implications for *reflection*.

## But arrays...
If generified lists aren't allowed to up-cast, why arrays are?  
The reason is very simple: arrays don't erase their element type. At runtime, each array knows exactly which kind of element it should allow in. If you try this code:

```java

String[] stringArray = new String[1];
Object[] objectArray = stringArray; // Perfectly normal

```

The compiler allows you to up-cast. What if we try to put something in the string array, that is not a string?

<!--more-->

```java

objectArray[0] = 1; //1 is not a String -> ArrayStoreException

```

Yep, the array reacts to your wrong doing and it throws you an ArrayStoreException if you try to put anything that is not a String (or subtype). The array retains its complete type information in runtime.

![Venn diagram of arrays](/images/2014-12-02-java-generics-and-inheritance-part-2-VennArrayObjectArrayString.jpg)

So basically, while you can see an array as an instance of a super type, the array doesn't lose its type. The type of an array is not lost at runtime because it's reified (there's an object that represents it at runtime). It doesn't depend on the type of the variable it's assigned to. 
On the other hand, a list type does depend on the variable it's assigned to, and that's why you cannot up-cast generic types, so you don't change the type of contained elements. As lists don't reify their element type you could put whatever inside, and then the implicit casts of generics may break.

## Not everything is erased
Would all this be fixed if we retained the element type of List, as arrays do?  
If we could do that, then we could be consistent about the list type, no matter what. And we could check that the list contains elements of a single type.
In other words if we retained the type argument of a generified type in runtime, using reflection we could reject wrong elements.  
I dare say that if the type arguments were reified with the parameterized type and made available to the created instance, none of the counter-intuitive generics restrictions would be needed. Assigning a string list to an object list variable would be as natural as doing so for arrays.

**Bad news** is that instantiating a generic class with a type argument like 

```java

new ArrayList<String>()

```

Will not preserve its type argument at runtime, and there seems to be no interest in doing so.

**Good news** is that extending a generic class with a type argument like

```java

class MyStringList extends ArrayList<String>{
  List<Object> castToObjectList(){return (List)this;}
}

```

Does retain the type argument in runtime. So using reflection to check for valid additions to the list is possible and also this kind of code is compile time safe:

```java

MyStringList stringList = new MyStringList();
List<Object> objectList = stringList.castToObjectList(); // no error
objectList.add(1);   // Proper error if not a string

```

## Multiple ways of getting the supertype
So far this has been an introduction to finally talk about reflection and generics.  
We declared a class that parametrizes a List:

```java

class MyStringList extends ArrayList<String>{
 ...
    public boolean add(Object s) {
        if(!getReifiedElementType().isInstance(s)){
            return false; // Or throw new IllegalArgument()
        }
        return super.add((String)s);
    }
 ...
}

```

It extends ArrayList with a *String* type parameter, and checks for valid additions based on the type of contained elements.  
This can be done at runtime using reflection. But let's see how that `getReifiedElementType()` could get the String type parameter from its superclass.

**Pre-java 5**: original `getSuperclass()` method

```java

		Class<?> myRawSuperclass= MyStringList.class.getSuperclass();

```

However, that will only return you an ArrayList.class instance. No type argument information.
This is the runtime super-type of our class (`ArrayList`), but it's not the compile time super-type (`ArrayList<String>`).

**Post-Java 5**: `Class.getGenericSuperclass()`

```java

Type myGenericSuperClass = MyStringList.class.getGenericSuperclass();

```

You guessed right. This Type instance has the type arguments. However, it wouldn't let you access it unless you know what sub-typ of `Type` is:

```java

  ParameterizedType parameterizedSuperclass = (ParameterizedType) myGenericSuperClass;
  Type[] typeArguments = parameterizedSuperclass.getActualTypeArguments();

```

Yeah. You are forced to downcast it. `Type` is designed in such a way that has no behavior and it is useless by itself. You need to ask what kind of instance is before doing something with it, and then downcast it.
If you check the returned Type array for arguments, you will find String.class as the only contained element (again, only after downcasting it will you be able to do something useful). 

**Post-Java 8**: `Class.getAnnotatedSuperclass()`

```java

  AnnotatedType annotatedSuperclass = MyStringList.class.getAnnotatedSuperclass();
  Type typeThatWasAnnotated = annotatedSuperclass.getType();
  Annotation[] annotations = annotatedSuperclass.getAnnotations();

```

What's this AnnotatedType instance you ask?  
It's how Oracle engineers extended reflection to support annotations on types. Since Java 8, you can use annotations on any type declaration. My class can extend an annotated super type:

```java

public class MyStringList extends ArrayList<@NotNull String> {

```

So, basically every type can have annotations that specify the type more precisely (although they have no effect on runtime). And the way to know which annotations and which type arguments are used is by calling `getAnnotatedSuperclass()`.
But it won't help you much. It uses `Type` hierarchy to know the type arguments.

## Ok, let's use the latest
Not only you do have 3 different ways to know your super type, you also have three different non-polymorphic objects.  If you are like me for simplicity's sake, you will say ok, let's use the most complete one, and let's forget about the others (the older ones). Let's use `getAnnotatedSuperclass():AnnotatedType` right?

![Three options for super-types](/images/2014-12-02-java-generics-and-inheritance-part-2-threeSuperTypes.jpg)

But what if I have more than one inheritance level? What if I want to know an argument of the super type of my super type? Does AnnotatedType has a super class method as Class does? A super type of List<String> is Collection<String> can I get to it? **Can I traverse my generic lineage? Short answer: no.**  

An `AnnotatedType` doesn't have the concept of inheritance in any of its methods .... But ...   
`AnnotatedType::getType()` allows you to get a `Type` instance which doesn't have the concept of inheritance either.... But....   
If `Type` is an instance of `ParameterizedType`, then you have `ParameterizedType::getRawType()` that gives you an instance of `Class` and finally you can ask that class its super type again.

![Whot?](/images/2014-12-02-java-generics-and-inheritance-part-2-whot.jpg)

Let me say it again: from a `Class` instance you can have the annotated super type (to get the annotations) then get the generic `Type`, cast it to a parameterized type (to get the type arguments), ask its raw class and only then get the super super type. It can't be easier!

![Navigating to a super type](/images/2014-12-02-java-generics-and-inheritance-part-2-superSuperType.jpg)

Oh, one more thing. Each level you go up, you lose the actual type arguments. Why?  
Because you have to pass through a Class instance in order to go up the next level. `Class` knows nothing about type arguments.  
In our example, once you get to the super type of ArrayList you will get a type variable `E` instead of the actual type argument `String`. You will have to manually keep track of type parameters, variable names and type arguments to figure out which is which.  

I have no idea why they just didn't add the annotations and generics information in `Type` instead of adding another `AnnotatedType`, and chose to have 3 different representations (and two parallel class hierarchies) for type information. Someone should have noticed that something was going wrong when they had almost same class names!

## One last thing: contradiction
Don't be too harsh on them. Reflection is supposed to represent parts of the language. It's not necessarily complete (it doesn't have to represent everything) but it should be true and precise about the things it does.  

The way java reflection API has evolved to include generics information is the result of an unsolvable contradiction produced by erasure.  
Java language at compile time is not what Java language is at runtime. This may seem obvious, but it's a recent discovery for me: if you have erasure (or any other compilation transformation) then you have two languages.  
The constructs that the language has at compile time don't exist at runtime and that's why reflection has internal contradictions at runtime when it tries to represent compile time concepts. A `List` is not the same type at compile-time as `List<String>`, however at runtime, they are!

That's why at runtime the super class of `MyStringList` is `ArrayList`, not `ArrayList<String>`. That's why you lose the type arguments of a parameterized type when you have a Class.  
Because making the language erase part of itself at compile time generates two different languages. And if you call with the same name the runtime and the compile versions of a language constructs, you will have contradictions.  
Two different personalities that need one representation. That's why you end up with a schizo reflection. You are two inside one.