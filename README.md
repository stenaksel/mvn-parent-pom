# mvn-parent-pom
##Introduction
When working with Maven as a build system, you often end up copying configuration of dependencies, and use of  
different plugins from an old project pom.xml file to the pom.xml file in a new project.

Getting configuration and settings right can be time consuming so reuse is helpful.
A Maven best-practice solution is using a "parent POM" for your new project (instead of copying).
A Maven parent POM can collect common configuration and settings that shall be used by many projects in your company.  

This is a typical setup: 
```
<project>
  <parent>
    <groupId>org.shared</groupId>
    <artifactId>company-parent-pom</artifactId>
    <version>1.0-SNAPSHOT</version>
  </parent>
  
  <groupId>org.company</groupId>
  <artifactId>new-project-name</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>pom</packaging>
  <name>Shiny New Project</name>
```
 
Here the "new-project-name" pom.xml file will inherit and reuse everything from the "**company-parent-pom**" file.
It is not uncommon to have a *hierarchy* of parent and grand-parent POMs. 
Eg. each project pom inherits a team parent POM which in turn inherits a organisation parent POM. 

##Usage
**mvn-parent-pom** have the potential to be used both as a direct parent and as grand-parent - actually a top-parent POM 
for all others to inherit :-)
This example setup put it as a parent to the **company-parent-pom**:

```
<project>
  <parent>
    <groupId>org.shared</groupId>
    <artifactId>mvn-parent-pom</artifactId>
    <version>1.0-SNAPSHOT</version>
  </parent>
  
  <groupId>org.company</groupId>
  <artifactId>company-parent-pom</artifactId>
  <version>2.0.0-SNAPSHOT</version>
  <packaging>pom</packaging>
  <name>Company Parent POM</name>
```

The projects that uses the **company-parent-pom** as a parent POM will now have **mvn-parent-pom** as a *grand-parent*, and
all the useful profiles in the **mvn-parent-pom** can now be used by projects through this inheritance.

(Using the **mvn-parent-pom** as a direct parent have the same configuration as shown above.)

###Activating profiles you want used 

When using Maven you may manually-activate one or more build profiles. This can be done in many ways. 
Profiles can be activated in the Maven settings (your **~/.m2/settings.xml** file), via the *activeProfiles* section. 
This section takes a list of <activeProfile> elements, each containing a profile-id inside.
You may add a list of profiles like shown below:

````
Activate profiles by adding them to your in your ~/.m2/settings.xml file, like shown below:
	<activeProfiles>
		<activeProfile>About</activeProfile>
		<activeProfile>mvn-debug</activeProfile>
		<activeProfile>show-profiles-used</activeProfile>
		<activeProfile>show-version-info</activeProfile>
	</activeProfiles>
````
*  Activate the "mvn-debug" profile to get hints for the use and configuration of your Maven project.
*  Activate the "show-version-info" to be informed of new versions of dependencies and plugins.

The list of *"activeProfiles"* can be specified in the order in which they should be applied.

###Skipping use of default profile(s)

The only default profile configured is the About profile, but if you don't like to see the text 
from the About profile you may add the skip-about property to you POM file:
````
    <skip-about>true</skip-about>
````
The profile is no longer default activated.

Another way to reduce some debug output is to use the **"mvn-debug-reduced"** profile.

When creating your own profiles you should use the same pattern used in **mvn-parent-pom** of adding properties 
to help identify the used profiles. Let's say you create a profile called **"My-profile"**, then you should add 
a property to it called **"*profile-in-use_My-profile*"**. 

Then the name of your profile will be show when using the **"show-profiles-used"**. 
This profile will show all properties starting with "profile-in-use_" :-)

If you are planning to use a "*skip-profile* property" like this example from **"mvn-debug"** profile where the 
skip-mvn-debug property with value true will 

````
    <profile>
      <id>mvn-debug</id>
      <activation>
        <activeByDefault>false</activeByDefault>
        <property>
          <name>!skip-mvn-debug</name>
          <value>true</value>
        </property>
      </activation>
      ....

````

####Warning
Be aware that setting of properies in profiles can be tricky, 
especially when you override eksisting properties. Take care to only set/override a property one place. Think of the 
problem if you set/override the same property in multiple profiles, then you may loose control of which profile that
will put the value into the property (- last profile changing the propery wins). 
     
