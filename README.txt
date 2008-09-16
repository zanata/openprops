
OpenProps is a tiny Java library which reads and writes .properties files 
using the same code as java.util.Properties from the OpenJDK, but enhanced so
that it preserves the order of entries within the file, and it also preserves
comments in the file.  This means that a Properties editor or a file converter written to use 
OpenProps won't have to lose comments or mess up the order of entries. 

By using OpenJDK code, OpenProps should handle all the old corner-cases in 
exactly the same way Java does.  The handling of whitespace and comments is
tested by a number of JUnit tests.  But please let me know if you find a bug!

Note the following differences from java.util.Properties:

1. preserves comments and the order of entries in the file
2. storeToXml doesn't use the Sun DTD (or any DTD) because it adds attributes for comments.
3. equals() and hashCode() won't work the same way as with java.util.Properties, because
   they are no longer inherited from Hashtable.  All you get is identity equality/hashcode.

Also note that any header comment in the .properties file will be interpreted as
a comment attached to the first message.


Licence:

OpenProps is based on the OpenJDK source code for java.util.Properties, 
and uses the same licence: GPLv2 + Classpath Exception. 
(Specifically, Properties.java and XMLUtil.java are taken from
 java-1.6.0-openjdk-1.6.0.0-0.16.b09.fc9.src.rpm in Fedora 9.)


Source code notes:

The original versions of java.util.Properties and the helper class 
java.util.XMLUtils are in the "orig" directory, for reference in 
generating patches.

If you are wondering why OpenProps isn't just a patch to java.util.Properties, 
submitted for inclusion in the OpenJDK, it's because java.util.Properties 
extends Hashtable, and thus can't cleanly use another implementation (such as 
LinkedHashMap) without breaking an established Java API.  

Changelog
* Tue Sep 16 2008 Sean Flanigan <sflaniga@redhat.com>
- Initial release based on OpenJDK 1.6.0

