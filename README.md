# NullDefense | Gson
Removes invalid objects during Gson parsing which are marked as required, yet null/empty.

[![Build Status](https://img.shields.io/travis/VenomVendor/NullDefense/master.svg?logo=travis)](https://travis-ci.org/VenomVendor/NullDefense)
[![License Apache-v2.0](https://img.shields.io/badge/license-Apache--2.0-brightgreen.svg)](https://github.com/VenomVendor/NullDefense/blob/master/LICENSE)
[![Download](https://api.bintray.com/packages/venomvendor/maven/NullDefense/images/download.svg) ](https://bintray.com/venomvendor/maven/NullDefense/_latestVersion)

# Code Quality
[![Codecov](https://codecov.io/gh/VenomVendor/NullDefense/branch/master/graph/badge.svg)](https://codecov.io/gh/VenomVendor/NullDefense)
[![Codacy Grade](https://api.codacy.com/project/badge/Grade/f067e5c9a9c14c53843bc56f0669d993)](https://www.codacy.com/app/VenomVendor/NullDefense?utm_source=github.com&utm_medium=referral&utm_content=VenomVendor/NullDefense&utm_campaign=Badge_Grade)
[![Codacy Coverage](https://api.codacy.com/project/badge/Coverage/f067e5c9a9c14c53843bc56f0669d993)](https://www.codacy.com/app/VenomVendor/NullDefense?utm_source=github.com&utm_medium=referral&utm_content=VenomVendor/NullDefense&utm_campaign=Badge_Coverage)
[![Codebeat](https://codebeat.co/badges/ef5996c2-d284-454e-a497-f5438f8867e7)](https://codebeat.co/projects/github-com-venomvendor-nulldefense-master)
[![CodeFactor](https://www.codefactor.io/repository/github/venomvendor/nulldefense/badge)](https://www.codefactor.io/repository/github/venomvendor/nulldefense)
[![Codeclimate Maintainability](https://api.codeclimate.com/v1/badges/12788f6de414f39eb749/maintainability)](https://codeclimate.com/github/VenomVendor/NullDefense/maintainability)
[![Codeclimate Coverage](https://api.codeclimate.com/v1/badges/12788f6de414f39eb749/test_coverage)](https://codeclimate.com/github/VenomVendor/NullDefense/test_coverage)

-------------------

## Installation
In `build.gradle` add the following dependencies

```coffeescript
repositories {
    maven {
        url  'https://dl.bintray.com/venomvendor/maven'
    }
}

dependencies {
    compile 'com.venomvendor:gson-nulldefense:<latest.version>'
}
```

## Usage Guide

### Annotation
- Use any existing annotation with [RetentionPolicy.RUNTIME](https://docs.oracle.com/javase/7/docs/api/java/lang/annotation/RetentionPolicy.html)
- Creating simple annotation with [RetentionPolicy.RUNTIME](https://docs.oracle.com/javase/7/docs/api/java/lang/annotation/RetentionPolicy.html)

    ```java
    package com.example.annotation;
    
    import java.lang.annotation.ElementType;
    import java.lang.annotation.Retention;
    import java.lang.annotation.RetentionPolicy;
    import java.lang.annotation.Target;
    
    /* Retention Policy should be runtime */
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ElementType.FIELD})
    public @interface Mandatory {
    
    }
    ```
> Tip: If you are using [Realm in Android](https://github.com/realm/realm-java), use [@io.realm.annotations.Required](https://github.com/realm/realm-java/blob/master/realm-annotations/src/main/java/io/realm/annotations/Required.java) 


### Marking required fields
- Annotate in POJOs used with Gson
    ```java
    public class CustomerDetail {
    
        @Mandatory
        @SerializedName("first_name") private String firstName;

        // Optional
        @SerializedName("lastName") private String lastName;

        @Mandatory
        @SerializedName("addresses") private List<Address> addresses;

        ...
        ...
        ...

        // Setter/Getter
    }
    ```

### Registering Adapter
```java
TypeAdapterFactory nullDefenceAdapter = new NullDefenseTypeAdapterFactory(Mandatory.class);

Gson gson = new GsonBuilder()
        // Add null defense Adapter
        .registerTypeAdapterFactory(nullDefenceAdapter)
        // More options
        .enableComplexMapKeySerialization()
        .setPrettyPrinting()
        .setLenient()
        .serializeNulls()
        .create();
```

### To Retain Empty Collection
```java
TypeAdapterFactory typeAdapter = new NullDefenseTypeAdapterFactory(Mandatory.class)
         // To retain empty collection
         .retainEmptyCollection();
```

### To Remove Empty Collection
```java
TypeAdapterFactory typeAdapter = new NullDefenseTypeAdapterFactory(Mandatory.class)
         // To remove empty collection, this is default
         .removeEmptyCollection();
```

-------------------


# Tests

#### POJO

```java
public class Parent {

    @Mandatory
    private String name;

    @Mandatory
    private CustomList<Child> children;

    // Setter/Getter
}

public class Child {

    @Mandatory
    private String name;
    // I am optional
    private Boolean isMale;

    // Has no effect on primitive
    @Mandatory
    private int age;

    @Mandatory
    private Language language;

    // Setter/Getter
}

public class Language {

    @Mandatory
    private List<String> knownLanguages;

    // I am optional
    private List<String> learning;

    // Setter/Getter
}
```

#### parent-valid.json

```json
{
  "name": "VenomVendor",
  "children": [
    {
      "name": "Queen",
      "isMale": false,
      "age": 26,
      "language": {
        "knownLanguages": [
          "Telugu",
          "Tamil"
        ],
        "learning": [
          "French"
        ]
      }
    },
    {
      "name": "Prince",
      "isMale": true,
      "age": 5,
      "language": {
        "knownLanguages": [
          "English",
          "Sanskrit"
        ],
        "learning": []
      }
    },
    {
      "name": "Princess",
      "isMale": false,
      "age": 3,
      "language": {
        "knownLanguages": [
          "Kannada"
        ],
        "learning": null
      }
    }
  ]
}
```

##### Result
```java
assertEquals(3, parent.getChildren().size());
```

#### valid-missing-primitive.json

> Annotation has no effect on primitive fields

```json
{
  "name": "VenomVendor",
  "children": [
    {
      "name": "Queen",
      "isMale": false,
      "age": 26,
      "language": {
        "knownLanguages": [
          "Telugu",
          "Tamil"
        ],
        "learning": [
          "French"
        ]
      }
    },
    {
      "name": "Prince",
      "isMale": true,
      "age": null,
      "language": {
        "knownLanguages": [
          "English",
          "Sanskrit"
        ],
        "learning": []
      }
    },
    {
      "name": "Princess",
      "isMale": false,
      "language": {
        "knownLanguages": [
          "Kannada"
        ],
        "learning": null
      }
    }
  ]
}
```

##### Result
```java
assertEquals(3, parent.getChildren().size());
```

#### missing-name-in-child.json

> Should generate one child, as rest doesn't have mandatory `name`

```json
{
  "name": "VenomVendor",
  "children": [
    {
      /* Missing Mandatory Field */
      "isMale": false,
      "age": 26,
      "language": {
        "knownLanguages": [
          "Telugu",
          "Tamil"
        ],
        "learning": [
          "French"
        ]
      }
    },
    {
      "name": null, // Null Mandatory Field
      "isMale": true,
      "age": 5,
      "language": {
        "knownLanguages": [
          "English",
          "Sanskrit"
        ],
        "learning": []
      }
    },
    {
      "name": "Princess",
      "isMale": false,
      "age": 3,
      "language": {
        "knownLanguages": [
          "Kannada"
        ],
        "learning": null
      }
    }
  ]
}
```

##### Result
```java
assertEquals(1, parent.getChildren().size());
```

#### missing-known-language-no-child.json

> Should not generate any children due to missing `knownLanguages`, hence parent is also `null`, as it doesn't have mandatory children

```json
{
  "name": "VenomVendor",
  "children": [
    {
      "name": "Queen",
      "isMale": false,
      "age": 26,
      "language": {
        "knownLanguages": null,
        "learning": [
          "French"
        ]
      }
    },
    {
      "name": "Prince",
      "isMale": true,
      "age": 5,
      "language": {
        "knownLanguages": [],
        "learning": []
      }
    },
    {
      "name": "Princess",
      "isMale": false,
      "age": 3,
      "language": {
        "learning": null
      }
    }
  ]
}
```

##### Result
```java
assertNull(parent);
```

-------------------

<br/>
## More [Tests⬈](https://github.com/VenomVendor/NullDefense/tree/master/src/test "Unit Tests")<br/><br/>

## <a target="_blank" href="https://venomvendor.github.io/NullDefense/docs/javadoc/">Java Docs⬈<a/><br/><br/>

## Code Coverage [![Codecov](https://codecov.io/gh/VenomVendor/NullDefense/branch/master/graph/badge.svg)](https://codecov.io/gh/VenomVendor/NullDefense)

<a href="https://codecov.io/gh/VenomVendor/NullDefense/list/master/"><img src="https://codecov.io/gh/VenomVendor/NullDefense/branch/master/graphs/sunburst.svg" width="300" /><a/>
