Enterprise-ready production-ready SOAP-Webservices powered by Spring Boot & Apache CXF
======================================================================================
[![Build Status](https://travis-ci.org/codecentric/cxf-spring-boot-starter.svg?branch=master)](https://travis-ci.org/codecentric/cxf-spring-boot-starter)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/de.codecentric/cxf-spring-boot-starter/badge.svg)](https://maven-badges.herokuapp.com/maven-central/de.codecentric/cxf-spring-boot-starter/)
[![Dependency Status](https://www.versioneye.com/user/projects/5720e300fcd19a0051856aa7/badge.svg?style=flat)](https://www.versioneye.com/user/projects/5720e300fcd19a0051856aa7)
[![License](http://img.shields.io/:license-apache-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0.html)

> !! This document applies to the next version under development.
>
> &nbsp; &nbsp; See [here for documentation on the latest released version](https://github.com/codecentric/cxf-spring-boot-starter/tree/1.0.7.RELEASE).

# Features include:

* __Generating all necessary Java-Classes__ using JAX-B from your WSDL/XSDs (using the complementing Maven plugin [cxf-spring-boot-starter-maven-plugin]
* Booting up Apache CXF within Spring Context with __100% pure Java-Configuration__
* __Complete automation of Endpoint initialization__ - no need to configure Apache CXF Endpoints, that´s all done for you automatically based upon the WSDL and the generated Java-Classes (bringing up a nice [Spring Boot 1.4.x Failure Message](http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-spring-application.html#_startup_failure) if you missed something :) )
* Customize SOAP service URL and the title of the CXF generated Service site
* __Configures CXF to use slf4j__ and serve Logging-Interceptors, to log only the SOAP-Messages onto console
* __Extract the SoapMessages__ for processing in __ELK-Stack__, like [docker-elk](https://github.com/jonashackt/docker-elk)
* Tailor your own __custom SOAP faults__, that comply with the exceptions defined inside your XML schema
* SOAP Testing-Framework: With __XmlUtils__ to easy your work with JAX-B class handling & a __SOAP Raw Client__ to Test malformed XML against your Endpoints


# Documentation

There´s also an blog post describing this project: [Spring Boot & Apache CXF – SOAP on steroids fueled by cxf-spring-boot-starter](https://blog.codecentric.de/en/2016/10/spring-boot-apache-cxf-spring-boot-starter/)

### Initial Setup

* Create a Spring Boot maven project. Use the __spring-boot-starter-parent__ as a parent and the __spring-boot-maven-plugin__ as a build plugin (you could speed that up, if you use the [Spring Initializr](https://start.spring.io/)).
* Then append cxf-spring-boot-starter as dependency and the [cxf-spring-boot-starter-maven-plugin] as build-plugin (see the example [cxf-boot-simple]):

```
<dependencies>
	<dependency>
		<groupId>de.codecentric</groupId>
    	<artifactId>cxf-spring-boot-starter</artifactId>
    	<version>1.0.7.RELEASE</version>
	</dependency>
</dependencies>
```


```
<build>
    <plugins>
        <plugin>
            <groupId>de.codecentric</groupId>
            <artifactId>cxf-spring-boot-starter-maven-plugin</artifactId>
            <version>1.0.7.RELEASE</version>
            <executions>
				<execution>
					<goals>
						<goal>generate</goal>
					</goals>
				</execution>
			</executions>
        </plugin>
    </plugins>
</build>
```

> only temporarily!

* due to logback / logstash-logback-encoder issue downgrade Spring Boot logback dependency (see [Extract the SoapMessages for processing in ELK-Stack](#extract-the-soapmessages-for-processing-in-elk-stack))

```
 <properties>
    <!-- Overriding SpringBoot 1.3.4ff´s logback 1.1.7, because of http://jira.qos.ch/browse/LOGBACK-1164 which is only fixed in the unreleased 1.1.8  -->
    <logback.version>1.1.6</logback.version>
</properties>
```

* place your .wsdl-File (and all the imported XSDs) into a folder somewhere under __src/main/resources__ (see [cxf-spring-boot-starter-maven-plugin] for details)
* run __mvn generate-sources__ to generate all necessary Java-Classes from your WSDL/XSD 
* Implement the [javax.jws.WebService](http://docs.oracle.com/javaee/7/api/javax/jws/WebService.html) annotated Interface (your generated __Service Endpoint Interface (SEI)__ ) - it is the starting point for your development and is needed to autoconfigure your Endpoints. See the [WeatherServiceEndpoint](https://github.com/codecentric/spring-samples/blob/master/cxf-boot-simple/src/main/java/de/codecentric/soap/endpoint/WeatherServiceEndpoint.java) class inside the [cxf-boot-simple] project. 
* That´s it



# Additional Configuration Options

### Customize URL, where your SOAP services are published

* create a application.properties and set the BaseURL of your Webservices via __soap.service.base.url=/yourUrlHere__


### Customize title of generated CXF-site

* place a __cxf.servicelist.title=Your custom title here__ in application.properties


### SOAP-Message-Logging

Activate SOAP-Message-Logging just via Property __soap.messages.logging=true__ in application.properties (no more configuration on the Endpoint needed) 

SOAP-Messages will be logged only and printed onto STOUT/Console for fast analysis in development.

### Extract the SoapMessages for processing in ELK-Stack

> If you want to use this Logging feature to extract the SoapMessages for processing in ELK-Stack, there are two possible ways: stick to __Spring Boot 1.3.3.RELEASE__, due to [problems](https://github.com/logstash/logstash-logback-encoder/issues/160) in the [logstash-logback-encoder], which is itself a bug in [logback 1.1.7](http://jira.qos.ch/browse/LOGBACK-1164), which again is delivered with SpringBoot. This is only, till the SpringBoot guys [update to logback 1.1.8](https://github.com/spring-projects/spring-boot/issues/6214) - but this is not released [yet](https://mvnrepository.com/artifact/ch.qos.logback/logback-core).
Or as an alternative way - and if you want to use __newer SpringBoot versions__ - you have to override the logback-version with a 
```<logback.version>1.1.6</logback.version>``` in your Maven´s properties-tag.


The cxf-spring-boot-starter brings some nice features, you can use with an ELK-Stack to monitor your SOAP-Service-Calls:

* Extract SOAP-Service-Method for Loganalysis (based on WSDL 1.1 spec, 1.2 not supported for now - because this is read from the HTTP-Header field SoapAction, which isn´ mandatory in 1.2 any more)
* Dead simple Calltime-Logging
* Correlate all Log-Messages (Selfmade + ApacheCXFs SOAP-Messages) within the Scope of one Service-Consumer`s Call in Kibana via logback´s [MDC], placed in a Servlet-Filter

##### HowTo use

* Activate via Property __soap.messages.extract=true__ in application.properties
* Add a __logback-spring.xml__ file to __src/main/resources__ (otherwise the feature will not be activated) and configure the [logstash-logback-encoder] (which is delivered with this spring-boot-starter), like:

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>
    <logger name="org.springframework" level="WARN"/>
    <!-- more logging config here -->
    
    
    <!-- Logstash-Configuration -->
	<!-- For details see https://github.com/logstash/logstash-logback-encoder/tree/logstash-logback-encoder-4.5 -->
	<appender name="logstash" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
		<!-- You may want to configure a default instance, this could be done like
		<destination>${LOGANALYSIS_HOST:-192.168.99.100}:5000</destination> as discribed here:
		http://logback.qos.ch/manual/configuration.html#defaultValuesForVariables
		Set the SystemProperty with
		env LOGANALYSIS_HOST={{ loganalysis.host }}
		e.g. in a service.upstart.conf.j2, when using Ansible and deploying to Ubuntu -->
		<destination>192.168.99.100:5000</destination>
		<!-- encoder is required -->
	    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
		   	<includeCallerData>true</includeCallerData>
		   	<customFields>{"service_name":"WeatherService_1.0"}</customFields>
		   	<fieldNames>
		   		<message>log-msg</message>
		   	</fieldNames>
	   	</encoder>
	   	<keepAliveDuration>5 minutes</keepAliveDuration>
	</appender>
	
	<root level="INFO">
	    <appender-ref ref="logstash" />
	</root>
</configuration>
```

* Now some fields will become available in your kibana dashboard (in other words in your elasticsearch index), e.g. soap-message-inbound contains the Inbound Message
* see all of them here [ElasticsearchField.java](https://github.com/codecentric/cxf-spring-boot-starter/blob/master/src/main/java/de/codecentric/cxf/logging/ElasticsearchField.java)
* Additionally [Spring Cloud Sleuth](http://cloud.spring.io/spring-cloud-sleuth/) will provide detailed tracing information of your services. Sleuth will populate the Logback MDC automatically with the tracing information.
* The default is to use the ELK stack for log analysis. With further configuration you can even extend the tracing infrastructure to use more tailored tracing tools like [Zipkin](http://zipkin.io/).

### Custom SOAP faults for XML Schema Validation Errors

The standard behavior of Apache CXF with XML validation errors (non schema compliant XML or incorrect XML itself) is to return a SOAP fault including the corresponding exception in CXF:
```
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
   <soap:Body>
      <soap:Fault>
         <faultcode>soap:Server</faultcode>
         <faultstring>wrong number of arguments while invoking public de.codecentric.namespace.weatherservice.general.ForecastRequest de.codecentric.cxf.WeatherServiceEndpoint.getCityForecastByZIP(de.codecentric.namespace.weatherservice.general.ForecastRequest) throws net.bipro.namespace.BiproException with params null.</faultstring>
      </soap:Fault>
   </soap:Body>
</soap:Envelope>
```
Many SOAP based standards demand a custom SOAP-Fault, that should be delivered in case of XML validation errors. To Implement that behavior, you have to:
* Implement the Interface [CustomFaultBuilder](https://github.com/codecentric/cxf-spring-boot-starter/blob/master/src/main/java/de/codecentric/cxf/xmlvalidation/CustomFaultBuilder.java) as Spring
[@Component](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Component.html)
* Override Method createCustomFaultMessage(FaultType faultContent) an give back appropriate Messages you want to see in faultstring: <soap:Fault><faultstring>YOUR CUSTOM MESSAGE HERE</faultstring>
* Override Method createCustomFaultDetail(String originalFaultMessage, FaultType faultContent) and return the JAX-B generated Object, that represents your WebService´ Fault-Details (be really careful to take the right one!!, often the term 'Exception' is used twice... - e.g. with the [BiPro]-Services)
* Configure your Implementation as @Bean - only then, XML Schema Validation will be activated


### Testing SOAP web services

##### Create a WebService Client

* If you instantiate a JaxWsProxyFactoryBean, you need to set an Adress containing your configured (or the standard) soap.service.base.url. To get the correct path, just autowire the CxfAutoConfiguration like:

``` 
@Autowired
private CxfAutoConfiguration cxfAutoConfiguration;
```

and obtain the __base.url__ and the __serviceUrlEnding__ (this one is derived from the wsdl:service name attribute of your WSDL) by calling

```
cxfAutoConfiguration.getBaseAndServiceEndingUrl()
```

##### Integrate real XML test files into your Unit-, Integration- or SingleSystemIntegrationTests

As described [in this blogpost](https://blog.codecentric.de/en/2016/06/spring-boot-apache-cxf-testing-soap-webservices/) the best gut feeling one could get while writing SOAP Tests, is the usage of real XML test files. To easily marshall these into your Java classes with JAX-B, this starter brings a utility class [de.codecentric.cxf.common.XmlUtils](https://github.com/codecentric/cxf-spring-boot-starter/blob/master/src/main/java/de/codecentric/cxf/common/XmlUtils.java) with lots of useful methods like __readSoapMessageFromStreamAndUnmarshallBody2Object(java.io.InputStream fileStream, Class<T> jaxbClass)__. Then you could do things inside your testcases like:

```
@Value(value="classpath:requests/GetCityForecastByZIPTest.xml")
private org.springframework.core.io.Resource GetCityForecastByZIPTestXml;
    
@Test
public void getCityForecastByZIP() throws WeatherException, BootStarterCxfException, IOException {
    GetCityForecastByZIP getCityForecastByZIP = XmlUtils.readSoapMessageFromStreamAndUnmarshallBody2Object(GetCityForecastByZIPTestXml.getInputStream(), GetCityForecastByZIP.class);
...
}
```

##### SOAP Raw client

Enables automatic testing of malformed XML Requests (e.g. for Testing your Custom SOAP faults) with the [de.codecentric.cxf.soaprawclient.SoapRawClient](https://github.com/codecentric/cxf-spring-boot-starter/blob/master/src/main/java/de/codecentric/cxf/soaprawclient/SoapRawClient.java). To use it in your Testcases, initialize the SoapRawClient inside a @Configuration annotated Class like this:

```
@Bean
public SoapRawClient soapRawClient() throws BootStarterCxfException {
    return new SoapRawClient(buildUrl(), YourServiceInterface.class);
}

public String buildUrl() {
    // return something like http://localhost:8084/soap-api/WeatherSoapService
    return "http://localhost:8084" + cxfAutoConfiguration.getBaseAndServiceEndingUrl();
}

@Autowired
private CxfAutoConfiguration cxfAutoConfiguration;
```

# Concepts

### How does Complete automation of Endpoint initialization work?

If you start your Spring Boot application and everything went fine, then you should see some of those log messages inside your console:

```
[...] INFO 83684 --- [ost-startStop-1] d.c.c.a.WebServiceAutoDetector           : Found WebServiceClient class: 'de.codecentric.namespace.weatherservice.Weather'
[...] INFO 83684 --- [  restartedMain] d.c.c.a.WebServiceAutoDetector           : Found Service Endpoint Interface (SEI): 'de.codecentric.namespace.weatherservice.WeatherService'
[...] INFO 83684 --- [  restartedMain] d.c.c.a.WebServiceAutoDetector           : Found SEI implementing class: 'de.codecentric.soap.endpoint.WeatherServiceEndpoint'
```


# Contribution

If you want to know more or even contribute to this Spring Boot Starter, maybe you need some information like:
* [SpringBoot Docs - Creating your own auto-configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-auto-configuration.html)
* [The Missing piece - how @Conditional is also needed for your own custom spring boot starter](https://www.packtpub.com/books/content/writing-custom-spring-boot-starters)
* [Boot your own Infrastructure - JavaMagazin](https://public.centerdevice.de/a10fb484-49a8-4a70-ada9-5eeda8c69465)



[cxf-spring-boot-starter-maven-plugin: set wsdl directory]:(https://github.com/codecentric/cxf-spring-boot-starter-maven-plugin#set-wsdl-directory-optional)
[cxf-spring-boot-starter-maven-plugin]:https://github.com/codecentric/cxf-spring-boot-starter-maven-plugin
[BiPro]:https://bipro.net
[logstash-logback-encoder]:https://github.com/logstash/logstash-logback-encoder
[MDC]:http://logback.qos.ch/manual/mdc.html
[cxf-boot-simple]:https://github.com/codecentric/spring-samples/tree/master/cxf-boot-simple


