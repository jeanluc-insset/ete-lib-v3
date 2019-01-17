# ete-lib-v3
This project contains the v3 of ere-lib-impl, a library to process ETE pipelines

## Short description
Ete is an MDE tool written in Java.
It runs pipelines of actions on a model.
The model is written in MOF (a subset of UML). It must conform to the 2013-10-01 version of UML (2.4) and must be exported in an XMI file which conforms to the 2013-10-01 version of XMI.
These are the default values of MagicDraw™ 18.5. 
This project contains a stand-alone library which can be embedded in a Java program.
A convenient Maven plugin is provided as well.

## Prerequisites
The library needs Java ≥ 8

The Maven plugin needs Maven ≥ 3

## Installation
After downloading, run the install.sh script.

For Windows, no .bat script is provided. Instead, run the commands :
```
mvn install:install-file -Dfile=ete-lib-impl-3.1-jar-with-dependencies.jar -DgroupId=fr.insset.jeanluc -DartifactId=ete-lib-impl -Dversion=3.1 -Dpackaging=jar

mvn install:install-file -Dfile=ete-lib-impl-3.1-jar-with-dependencies.jar -DgroupId=fr.insset.jeanluc -DartifactId=ete-maven-plugin  -Dversion=3.1 -Dpackaging=maven-plugin
```
## Usage
Currently, we don't provide any documentation for embedding the library but for use in a Maven process.

### Adding the plugin to the project
Add the following declaration to the build/plugins section of the pom.xml file :
```xml
            <plugin>
                <groupId>fr.insset.jeanluc</groupId>
                <artifactId>ete-maven-plugin</artifactId>
                <version>3.1</version>
                <dependencies>
                    <dependency>
                        <groupId>fr.insset.jeanluc</groupId>
                        <artifactId>el-evaluator</artifactId>
                        <version>3.1</version>
                    </dependency>
                </dependencies>
                <executions>
                    <execution>
                        <id>ete</id>
                        <goals>
                            <goal>ete</goal>
                        </goals>
                        <phase>generate-sources</phase>
                    </execution>
                </executions>
            </plugin>
```

### Configuring the process
As previously stated, the process runs actions in a "pipeline".
The pipeline is configured in the src/main/mda/ete-config.xml file.
Here is a very simple example :
```xml
<?xml version="1.0" encoding="UTF-8"?>
<mda>
    <transformation-set>
        <velocity template="class2interface.vm"
                    items="${classes}"
                    target='${project}-api/target/generated-sources/ete/${current.owningPackage.name.replaceAll("\\.", "/")}/${prefix}${current.name}.java'/>
        <velocity template="enumeration.vm"
                    items="${enumerations}"
                    target='${project}-api/target/generated-sources/ete/${current.owningPackage.name.replaceAll("\\.", "/")}/${prefix}${current.name}.java'/>
        <velocity template="class2class.vm"
                    items="${classes}"
                    target='${project}-impl/target/generated-sources/ete/${current.owningPackage.name.replaceAll("\\.", "/")}/${prefix}${current.name}Impl.java'/>
        <velocity template="factory.vm"
                    target='${project}-impl/target/generated-sources/ete/fr/upjv/mis/ete/${dialect.i2lc(prefix)}/${prefix}Factories.java'/>
                    <!-- The optional classes allow smart navigation through eventually null references -->
        <velocity template="class2optional.vm"
                    items="${classes}"
                    target='${project}-api/target/generated-sources/ete/${current.owningPackage.name.replaceAll("\\.", "/")}/${prefix}${current.name}Optional.java'/>
    </transformation-set>
</mda>
```

### Velocity action
The velocity action applies a template written in Velocity template language to a collection of items from the model.
An item, a generated file.
The action takes two or three parameters :
  - the template to run
  - the file(s) to generate
  - (optionnally) the items the template must be applied on. If the items parameter is not provided, the model itself is the single item

For advance usage it is possible to provide additional parameters.
Any additional parameter is put into the Velocity context.

### Velocity template
  The $current variable points to the processed item.
  The $dialect variable points to an object which provides some utiliy methods.
  
  Here is an example of such a template :
  
```java
package fr.upjv.mis.ete.${dialect.i2lc($prefix)};

import fr.insset.jeanluc.util.factory.FactoryMethods;
import fr.insset.jeanluc.util.factory.FactoryRegistry;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.stream.Stream;
import javax.annotation.Generated;

@Generated("ETE core/src/main/mde/spec/factory.vm")
public class ${prefix}Factories {

    public static void initFactories() {
        FactoryRegistry registry = FactoryRegistry.getRegistry();
#foreach ($aClass in $current.classes)
#if (!$aClass.hasStereotype("ignore"))
        registry.registerDefaultFactory(${dialect.getQualifiedName($aClass)}.${aClass.name.toUpperCase()}, ${prefix}${aClass.name}Impl.class);
        registry.registerDefaultFactory(${dialect.getQualifiedName($aClass)}.class, ${prefix}${aClass.name}Impl.class);
#end
#end
    }
}
```
In this example, the template is applied to the whole model.
It contains a loop through the classes of that model.

The $prefix variable is set through an attribute of the Velocity action.


### More detailed information
Please stay tuned
