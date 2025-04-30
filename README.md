<a id="logo" href="https://www.paymentcomponents.com" title="Payment Components" target="_blank">
    <img loading="lazy" src="https://i.postimg.cc/yN5TNy29/LOGO-HORIZONTAL2.png" alt="Payment Components">
</a>

# Message Translator MEPS Demo

The project is here to demonstrate how our [SDK](https://www.paymentcomponents.com/messaging-libraries/) for MEPS
Message Translator works. For our demonstration we are going to use the demo SDK which can translate MT to MEPS
messages.

This documentation describes how to incorporate the MEPS Translator Library into your project. The SDK is written in
Java.
By following this guide you will be able to translate MT(ISO 15022) messages to MEPS messages
and vice versa according to MEPS guidelines.

It's a simple maven project, you can download it and run it, with Java 1.8 or above.

## SDK setup

Incorporate the
SDK [jar](https://nexus.paymentcomponents.com/repository/public/gr/datamation/translator-meps/4.43.1/translator-meps-4.43.1-demo.jar)
into your project by the regular IDE means.
This process will vary depending upon your specific IDE and you should consult your documentation on how to deploy a
bean.
For example in Intellij all that needs to be done is to import the jar files into a project. Alternatively, you can
import it as a Maven or Gradle dependency.

### Maven

Define repository in the repositories section

```xml

<repository>
<id>paymentcomponents</id>
<url>https://nexus.paymentcomponents.com/repository/public</url>
</repository>
```

Import the SDK

```xml

<dependency>
<groupId>gr.datamation</groupId>
<artifactId>translator-meps</artifactId>
<version>4.43.1</version>
<classifier>demo</classifier>
</dependency>
```

Import additional dependencies if not included in your project

```xml

<dependency>
<groupId>org.codehaus.groovy</groupId>
<artifactId>groovy-all</artifactId>
<version>2.5.11</version>
<scope>compile</scope>
<type>pom</type>
</dependency>
```

For Java 11 or above

```xml

<dependency>
<groupId>org.codehaus.mojo</groupId>
<artifactId>jaxb2-maven-plugin</artifactId>
<version>2.5.0</version>
</dependency>
```

### Gradle

Define repository in the repositories section

```groovy
repositories {
maven {
url "https://nexus.paymentcomponents.com/repository/public"
}
}
```

Import the SDK

```groovy
implementation 'gr.datamation:translator-meps:4.43.1:demo@jar'
```

Import additional dependencies if not included in your project

```groovy
implementation group: 'org.codehaus.groovy', name: 'groovy-all', version: '2.5.11', ext: 'pom'
```

For Java 11 or above

```groovy
implementation group: 'org.codehaus.mojo', name: 'jaxb2-maven-plugin', version: '2.5.0'
```

## Supported MT > MX Translations

| MT message    | MX message           | Translator Class    | Available in Demo | Custom Translation Rules |
|---------------|----------------------|---------------------|:-----------------:|:------------------------:|
| MT202         | pacs.009.001.08.core | Mt202Mt205ToPacs009 |      &check;      |                          |

## Instructions

### Auto Translation

You have the option to provide the MT or MEPS message and the library auto translates it to its equivalent.
Input should be in text format.
Output could be either in text format or an Object.
You need to call the following static methods of `MepsTranslator` class.
In case of no error of the input message, you will get the translated message.
Translated message is not validated.

java
```
public static String translateMtToMx(String mtMessage) throws InvalidMtMessageException, StopTranslationException, TranslationUnhandledException

public static MepsMessage translateMtToMxObject(String mtMessage) throws InvalidMtMessageException, StopTranslationException, TranslationUnhandledException
```

java
```
public static String translateMxToMt(String mxMessage) throws InvalidMxMessageException, StopTranslationException, TranslationUnhandledException

public static SwiftMessage translateMxToMtObject(String mxMessage) throws InvalidMxMessageException, StopTranslationException, TranslationUnhandledException
```

### Explicit Translation

If you do not want to use the auto-translation functionality, you can call directly the Translator you want.
        In this case you need to know the exact translation mapping.
Translator classes implement the `MtToMepsTranslator` or `MepsToMtTranslator` interface.
The `translate(Object)`, does not validate the message.
        The `translate(String)`, validates the message.
Translated message is not validated.

`MtToMepsTranslator` interface provides the following methods for both text and object format translations.

```java
String translate(String swiftMtMessageText) throws InvalidMtMessageException, StopTranslationException, TranslationUnhandledException;

MepsMessage translate(SwiftMessage swiftMtMessage) throws StopTranslationException, TranslationUnhandledException;
```

`MepsToMtTranslator` interface provides the following methods.

```java
String translate(String mepsMessageText) throws InvalidMxMessageException, StopTranslationException, TranslationUnhandledException;

SwiftMessage translate(MepsMessage mepsMessage) throws StopTranslationException, TranslationUnhandledException;

SwiftMessage[] translateMultipleMt(MepsMessage mepsMessage) throws StopTranslationException, TranslationUnhandledException;
```

The method `translateMultipleMt` translates a MEPS message to multiple MT messages. Until now, no translation uses this
method.

Both `MtToMepsTranslator` and `MepsToMtTranslator` interfaces provide a method to get the translation error list.

```java
List<TranslationError> getTranslationErrorList();
```

For the structure of `TranslationError` see [Error Handling](#translation-error-list).

In case that a translation uses this logic, the translation in text format will return the MT messages splitted
with `$`.
For example:

```
:56A:INTERBIC
-}${1:F01TESTBICAXXXX1111111111}
```

### Message Validation

In order to validate the translated MT message, you can
use `MtMessageValidationUtils.validateMtMessage(SwiftMessage mtMessage)`,
`MtMessageValidationUtils.parseAndValidateMtMessage(String message)` or any other way you prefer.
In order to validate the translated MEPS message, you can
use `MepsMessageValidationUtils.parseAndValidateMepsMessage(A appHdr, D documentMessage, MepsMessage.MepsMsgType mepsXsd)`
where
`MepsMsgType` can be retrieved from
MepsMessage.extractMepsMsgType(), `MepsMessageValidationUtils.autoParseAndValidateMepsMessage(String message)` or any
other way you prefer.

### Error Handling

#### InvalidMtMessageException & InvalidMxMessageException

When you translate a message, input message is validated. For example, in a MT→MX translation, the
first step is to validate the MT message and we proceed to translation only if the message is valid.
This is the reason why this direction throws `InvalidMtMessageException`.
The other direction throws `InvalidMxMessageException`.
Both Exceptions contain a `validationErrorList` attribute which contains a description of the errors occurred.

#### StopTranslationException

When there is a condition in input message that obstructs the translation, a `StopTranslationException` is thrown which
contains a `translationErrorList`. The `TranslationError` has the structure:

- errorCode
- errorCategory
- errorDescription

#### TranslationUnhandledException

When there is an exception that is not known, like `NullPointerException`, an `TranslationUnhandledException` is thrown
the
actual exception is attached as the cause.

### Modify the generated message

Once you have the translated message in text format, you can use our other Financial Messaging
Libraries ([Other Resources](#other-resources)) in order to create a Java Object and make any changes you want.

In order to create an MEPS Java Object from text use the below code. The class `FIToFICustomerCreditTransfer08` may
vary depending on the ISO20022 Message Type.
_Other message types are
available [here](https://github.com/Payment-Components/demo-iso20022#supported-meps-message-types)_

```java
MepsMessage<BusinessApplicationHeader02, FIToFICustomerCreditTransfer08> mepsMessage = new MepsMessage(new BusinessApplicationHeader02(), new FIToFICustomerCreditTransfer08());
mepsMessage.

parseXml(translatedMessageText);

BusinessApplicationHeader02 businessApplicationHeader02 = mepsMessage.getAppHdr();
FIToFICustomerCreditTransfer08 fiToFICustomerCreditTransfer = mepsMessage.getDocument();

//In case you want to enclose the MEPS message under another Root Element, use the code below
mepsMessage.

encloseMepsMessage("RequestPayload"); //In case you want RequestPayload
```

In order to create an MT Java Object use the below code.

```java
SwiftMessage swiftMessage = new SwiftMsgProcessor().ParseMsgStringToObject(translatedMessageText);
```

### Code Samples

In this project you can see code for all the basic manipulation of an MT or MEPS message, like:

- [Translate MT to MX](src/main/java/com/paymentcomponents/converter/meps/demo/TranslateMtToMx.java)
- [Translate MX to MT](src/main/java/com/paymentcomponents/converter/meps/demo/TranslateMxToMt.java)

### Other resources

- More information about our implementation of **MT library** can be found in our demo
on [PaymentComponents GitHub](https://github.com/Payment-Components/demo-swift-mt).
        - More information about our implementation of **ISO20022 library** can be found in our demo
on [PaymentComponents GitHub](https://github.com/Payment-Components/demo-iso20022).
        - More information about our implementation of **MEPS library** can be found in our demo
on [PaymentComponents GitHub](https://github.com/Payment-Components/demo-iso20022#meps-messages).
