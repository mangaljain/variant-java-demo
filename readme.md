![Variant Logo](http://www.getvariant.com/wp-content/uploads/2016/07/VariantLogoSquare-100.png)

# Variant Experience Server Demo Application
### Release 0.9.3
#### Requires: 
* [Variant Java Client 0.9.3](https://www.getvariant.com/resources/docs/0-9/clients/variant-java-client/)
* [Variant Experience Server 0.9.x](http://www.getvariant.com/resources/docs/0-9/experience-server/user-guide/) 
* Java 8 or later.

This __Variant Demo Application__ demonstrates instrumentation of two simple experience variations: an experiment (A/B test) and a concurrent feature toggle on a Java servlet Web application. This demonstration will help you
* Download, install and deploy Variant Experience Server on your local system;
* Clone and deploy this demo application on your local system;
* Step through the instrumented variations;
* Understand the instrumentation details of the demo.

Note, that this demo application is built with the popular [Pet Clinic webapp](https://github.com/spring-projects/spring-petclinic), available from the Sprinig MVC project. We are using it to demonstrate Variant's [Java client](http://getvariant.com/resources/docs/0-9/clients/variant-java-client) as well as the [servlet adapter for Variant Java client](https://github.com/getvariant/variant-java-servlet-adapter). If your application does not run in a servlet container, much of this demonstration will still be applicable.

## 1. Start Variant Server

• [Download and install](https://www.getvariant.com/resources/docs/0-9/experience-server/reference/#section-1) Variant Experience Server.

• Start Variant server:
```
% /path/to/server/bin/variant.sh start
```

If all went well, the server console output should look something like this:
```
[info] 19:10:12.717 c.v.c.c.ConfigLoader - Found  config resource [/variant.conf] as [/private/tmp/demo/variant-server-0.9.3/conf/variant.conf]
[info] 19:10:14.091 c.v.s.s.SchemaDeployerFileSystem - Mounted schemata directory [/private/tmp/demo/variant-server-0.9.3/schemata]
[info] 19:10:14.092 c.v.s.s.SchemaDeployerFileSystem - Deploying schema from file [/private/tmp/demo/variant-server-0.9.3/schemata/petclinic.schema]
[info] 19:10:14.285 c.v.s.s.ServerFlusherService - Registered event logger [com.variant.server.api.EventFlusherAppLogger] for schema [petclinic]
[info] 19:10:14.312 c.v.s.s.SchemaDeployerFileSystem - Deployed schema [petclinic] ID [38EFB1D4B56FCA01], from [petclinic.schema]:
   NewOwnerTest:[outOfTheBox (control), tosCheckbox, tosAndMailCheckbox] (ON)
[info] 19:10:14.317 c.v.s.b.VariantServerImpl - [431] Variant Experiment Server release 0.9.3 bootstrapped on :5377/variant in 00:01.247
```

Note, that Variant server comes pre-configured to run the demo application out-of-the-box. The `/schemata` directory contains the demo experiment schema file `petclinic.schema`, and the `/ext` directory contains the `server-extensions-demo-<release>.jar` file, containing the user hooks, used this demonstration.

## 2. Deploy the Demo Appliction

• Clone This Repository:
```
% git clone https://github.com/getvariant/variant-java-demo.git
```
• Change directory to `variant-java-demo`
```
% cd variant-java-demo
```
• Install Maven Dependencies

Variant Demo application is built on top of the [servlet adapter](https://github.com/getvariant/variant-java-servlet-adapter). It is included in this repository's `/lib` directory and must be installed in your local Maven repository: 

```
% mvn install:install-file -Dfile=lib/variant-java-client-0.9.3.jar -DgroupId=com.variant -DartifactId=variant-java-client -Dversion=0.9.3 -Dpackaging=jar

% mvn install:install-file -Dfile=lib/variant-core-0.9.3.jar -DgroupId=com.variant -DartifactId=variant-core -Dversion=0.9.3 -Dpackaging=jar

% mvn install:install-file -Dfile=lib/variant-java-client-servlet-adapter-0.9.3.jar -DgroupId=com.variant -DartifactId=variant-java-client-servlet-adapter -Dversion=0.9.3 -Dpackaging=jar
```

• Start the demo application:
```
% mvn tomcat7:run
```
If all went well, you will see the following console output:
```
INFO  2017-08-03 16:46:42 VariantConfigLoader - Found config resource [/variant.conf] as [/private/tmp/demo/variant-java-servlet-adapter/servlet-adapter-demo/target/classes/variant.conf]
INFO  2017-08-03 16:46:43 VariantFilter - Connected to schema [petclinic]
```
The demo application is accessible at <span class="variant-code">http://localhost:9966/petclinic/</span>.

• Optionally, configure a custom Variant server URL

By default, the demo application looks for Variant server at the default URL `http://localhost:5377/variant`. If your server is running elsewhere, you must update the Variant client configuration file [/src/main/resources/variant.conf](https://github.com/getvariant/variant-java-demo/blob/master/src/main/resources/variant.conf) by setting the `server.url` property. Restart Variant server to have the new value take effect.


## 3. Run the Demo

### 3.1 User Experiences

The demo consists of two variations:

__The feature toggle [`VetsHourlyRateFeature`](https://github.com/getvariant/variant-java-demo/blob/9affd4cc3992e8adf109a79532a1de75764ea38f/petclinic.schema#L44-L74)__ exposes an early release of a the new feature on the `Veterinarians` page, which adds the _Hourly Rate_ column to the table.

__The experiment [`ScheduleVisitTest`](https://github.com/getvariant/variant-java-demo/blob/9affd4cc3992e8adf109a79532a1de75764ea38f/petclinic.schema#L80-L142)__ validates another new feature, designed to improve new appointment bookings by displaying new _Availability_ column on the same same `Veterinarians` page. 

Since the  `Veterinarians` page is shared by both variations, it can have 4 variants:

<table>
  <tr>
    <th>VetsHourlyRate
       Feature</th>
    <th colspan="2">ScheduleVisitTest</th>
  </tr>
  <tr>
    <td>&nbsp;</td>
    <td>Control</td>
    <td>With Availability Column</td>
  </tr>
  <tr>
    <td>Control</td>
    <td>
       <img src="https://github.com/getvariant/variant-java-demo/blob/767b758b2e145dca688bbc65e521e5ac804f4fb7/docs/img/Fig-1-all-control.png">
       Existing code path.
     </td>
    <td>
        <img src="https://github.com/getvariant/variant-java-demo/blob/80424d4268bd6445569f05c2b3e0a431c784c14f/docs/img/Fig-1-with-appt-link.png">
       With availability column. (Proper state variant).
     </td>
  </tr>
  <tr>
    <td>With Hourly Rate Column</td>
    <td>
       <img src="https://github.com/getvariant/variant-java-demo/blob/80424d4268bd6445569f05c2b3e0a431c784c14f/docs/img/Fig-1-with-hourly-rate.png">
       With hourly rate column. (Proper state variant).
     </td>
    <td>
       <img src="https://github.com/getvariant/variant-java-demo/blob/80424d4268bd6445569f05c2b3e0a431c784c14f/docs/img/Fig-1-hybrid.png">
       With both columns. (Hybrid state variant).
     </td>
  </tr>
</table>

If a session is targeted for control experiences in both variations, it is served the existing `Veterinarians` page with two columns. If a session is targeted for a variant experience in either variation, and to control in the other, it sees either of the two _proper_ variants of the `Veterinarians` page with one extra column. Finally, if a session is targeted for variant experiences in _both_ variations, it is served the _hybrid_ variant of the `Veterinarians` page with two extra columns. 

Hybrid state variants are optional in Variant: unless explicitly configured in the schema, concurrent variations are treated as _disjoint_, i.e. Variant server will not target any sessions to variant experiences in both variations. However, in this demo, the more complex _conjoint_ concurrency model is demonstrated. It supports hybrid state variants, when both variations are targeted to the variant experience. This is explained in detain in a subsequent section.

When you first navigate to the `Veterinarians` page, Variant server targets your session randomly in both variations. This targeting is _durable_, so reloading the page won't change it. If you want to make Variant to re-target, get a new private browser window. (Note that some browsers share cookies between private windows, so be sure that there are no other private windows open.) You may also vary the probability weights ([here](https://github.com/getvariant/variant-java-demo/blob/ad4d55d1d7017c64e4e9c8248f7700eff2c32b61/petclinic.schema#L54) and [here](https://github.com/getvariant/variant-java-demo/blob/ad4d55d1d7017c64e4e9c8248f7700eff2c32b61/petclinic.schema#L91)) by editing the `petclinic.schema` file in the server's `schemata` directory.

If your session happened to be targeted to the variant experience in the `ScheduleVisitTest` and you see the _Schedule visit_ link, clicking it will navigate to the experimental version of the `New Visit`page, containing the vet's name:

<table>
  <tr>
    <th colspan="2">ScheduleVisitTest</th>
  </tr>
  <tr>
    <td>Control</td>
    <td>With Availability Column</td>
  </tr>
  <tr>
    <td>
       <img src="https://github.com/getvariant/variant-java-demo/blob/9a1f7df67fd559a6c4b83fab9c8567d09e678246/docs/img/Fig-2-control.png">
       Existing code path.
     </td>
    <td>
       <img src="https://github.com/getvariant/variant-java-demo/blob/9a1f7df67fd559a6c4b83fab9c8567d09e678246/docs/img/Fig-2-with-appt-link.png">
       With availability column.
     </td>
  </tr>
</table>

### 3.2 Event Tracing

The variation schema for this demo does not specify a tracing event flusher, deferring to the default [EventFlusherAppLogger](https://www.getvariant.com/javadoc/0.9/com/variant/server/api/EventFlusherAppLogger.html), which appends trace events to the server loger file `log/variant.log`:  

```
[info] 20:03:19.431 c.v.s.e.EventWriter$FlusherThread - Flushed 1 event(s) in 00:00:00.000
[info] 20:03:40.444 c.v.s.a.EventFlusherAppLogger - {event_name:'$STATE_VISIT', created_on:'1538967820228', session_id:'11BAEABC9F08B6F8', event_experiences:[{test_name:'ScheduleVisitTest', experience_name:'noLink', is_control:true}], event_attributes:[{key:'$STATE', value:'vets'}, {key:'HTTP_STATUS', value:'200'}, {key:'$STATUS', value:'Committed'}]}
```

The `STATE-VISIT` event is automatically triggered by Variant each time a user session visits an instrumented Web page. Note the delay between the time your session lands on an instrumented page and when the event is flushed to the log. This is due to the asynchronous nature of Variant's event writer. You can reduce the lag by changing the value of the `event.writer.max.delay` server configuration parameter in the `conf/variant.conf` file. 

You can also [configure a different trace event flusher](http://getvariant.com/resources/docs/0-9/experience-server/reference/#section-4.3) to utilize a persistence mechanism of your choice. 

Trace events provide the basis for analyzing variations. Features can be programmatically disabled if trace events indicate an unexpected failure, and experiments can be analyzed for target metrics and statistical significance.

## 4. Instrumentation

### 4.1 Variation Schema

The variation schema used by this demo application is included with the Variant server distribution, in the `schemata/petclinic.schema` file. For convenience, it is also replicated in this repo in [`petclinic.schemata`](https://github.com/getvariant/variant-java-demo/blob/06c66e40ea206609fc4ff370bce13998c2ea12b9/petclinic.schema).

The two states [`vets`](https://github.com/getvariant/variant-java-demo/blob/06c66e40ea206609fc4ff370bce13998c2ea12b9/petclinic.schema#L17-L25) and [`newVisit`](https://github.com/getvariant/variant-java-demo/blob/06c66e40ea206609fc4ff370bce13998c2ea12b9/petclinic.schema#L27-L35) correspond to the `Veterinarians` page and the `New Visit` page. The `path` state parameter is used by [the servlet adapter](https://github.com/getvariant/variant-java-servlet-adapter) to intercept the incoming HTTP request for these two pages.  

The [`VetsHourlyRateFeature`](https://github.com/getvariant/variant-java-demo/blob/b683f4c400d62d96f6e1539a34ead1df407f69de/petclinic.schema#L44-L69) variation is instrumented on the single `Veterinarians` page and has two experiences `existing` (control) and `rateColumn` with randomized weights 3:1 in favor of the variant. Note also the [`ChromeTargeter`](https://github.com/getvariant/variant-server-extapi/blob/master/src/main/java/com/variant/server/ext/demo/ChromeTargetingHook.java) lifecycle event hook which overrides the default randomized targeting by assigning all Chrome traffic to the control experience.


The [`ScheduleVisitTest`](https://github.com/getvariant/variant-java-demo/blob/b683f4c400d62d96f6e1539a34ead1df407f69de/petclinic.schema#L75-L137) variation is instrumented on two pages, starting on the `Veterinarians` page and ending on the `New Visit` page. Note the [`conjointVariationsRefs`](https://github.com/getvariant/variant-java-demo/blob/b683f4c400d62d96f6e1539a34ead1df407f69de/petclinic.schema#L77) specification, declaring the conjoint concurrence between the two variations. This specification tells Variant that 'ScheduleVisitTest` and `VetsHourlyReateFeature` are conjointly concurrent, i.e. that it's okay for Variant server to target a session for these two variations completely independently.

Note the [two state variants](https://github.com/getvariant/variant-java-demo/blob/b683f4c400d62d96f6e1539a34ead1df407f69de/petclinic.schema#L93-L120) on the `Veterinarians` page. They are needed to provide the new values of the `path` state parameter, specific to the two state variants. These values are utilized by the [`Variant servlet adapter`](https://github.com/getvariant/variant-java-servlet-adapter) to forward an incoming HTTP request to an alternate resource path.

### 4.2 Variant Servlet Adapter

This demo application makes extensive use of the [servlet adapter](https://github.com/getvariant/variant-java-servlet-adapter) for the Variant Java client. It is bootstrapped via the application's [deployment descriptor](https://github.com/getvariant/variant-java-demo/blob/327520377791b8abba0da2c3d8fbbd61473a04d2/src/main/webapp/WEB-INF/web.xml#L124-L137). The underlying [`VariantFilter`](https://getvariant.github.io/variant-java-servlet-adapter/com/variant/client/servlet/VariantFilter.html) intercepts all incoming HTTP request and matches their paths with those given in the [`path` state parameters](https://github.com/getvariant/variant-java-demo/blob/72f7460aa2f9cf2dc4f2922814c91650fb06d975/petclinic.schema#L18-L35). If the request path matches the value specified in a state, `VariantFilter` treats the current request as hitting an instrumented state. It then retrieves the value of the `path` state parameter, specified at the state variant level. If the state variant for which the current session is targeted, specifies a different value for the `path` state parameter, `VariantFilter` forwards the request to that resource path. Otherwise, `VariantFilter` lets the request through.

`VariantFilter` also declares [some callback methods](https://github.com/getvariant/variant-java-servlet-adapter/blob/c7e013cedc17f7d3dfbf4d408ed1c8f8dcf93594/src/main/java/com/variant/client/servlet/VariantFilter.java#L130-L144) that allow subclasses to inject additional semantics. In particular, the `onSession()` method is [taken advantage of](https://github.com/getvariant/variant-java-demo/blob/4d471e41e2dfccc480803cbf00b3c069233a5810/src/main/java/com/variant/client/servlet/demo/PetclinicVariantFilter.java#L12-L19) in this demo. Here we create a Variant session attribute `user-agent` holding the value of the `User-Agent` HTTP request header. This session attribute will be used by the `ChromeTargeter` lifecycle hook, as explained in the next section.

### 4.3 `ChromeTargeter` Lifecycle Hook 

The schema definition for the `VetsHourlyRateFeature` contains the [`ChromeTargeter`](https://github.com/getvariant/variant-server-extapi/blob/master/src/main/java/com/variant/server/ext/demo/ChromeTargetingHook.java) lifecycle event hook which overrides the default randomized targeting by assigning all Chrome traffic to the control experience. In a real application you will likely use feature flags to  roll out a new feature to a gradually increasing user population, e.g. by the zip code. This is best achieved though the use of a similar lifecycle hook, which operates on the server side, is highly reusable and entirely outside the host application's code base.

### 4.4 Experience Implementations

The `VetsHourlyRateFeature` variation does not specify any state parameter overrides, so if the session was targeted for the control experience in the covariant `ScheduleVisitTest` variation, `VariantFilter` lets the HTTP request fall through. Both experiences will proceed though the existing code path up until [`vets.jsp`](https://github.com/getvariant/variant-java-demo/blob/e783434ae9c5166d7c19e8ee6642714e12c6418d/src/main/webapp/WEB-INF/jsp/vets/vetList.jsp#L34-L54), where the new _Hourly Rate_ column is optionally displayed, depending on the live experience in effect. This type of experience instrumentation, where the code bifurcates as late in the code path as possible, is referred to as _lazy instrumentation_.

Alternatively, the `ScheduleVisitTest` variation makes use of the state parameter overrides. If a session is targeted to the `withLink` experience, `VariantFilter` forwards it to the new `/vets__ScheduleVisit_withLink.html` path, which is [mapped](https://github.com/getvariant/variant-java-demo/blob/e783434ae9c5166d7c19e8ee6642714e12c6418d/src/main/java/org/springframework/samples/petclinic/web/VetController.java#L54-L63) to the [`vets/vetList__ScheduleVisit_withLink.jsp`](https://github.com/getvariant/variant-java-demo/blob/e783434ae9c5166d7c19e8ee6642714e12c6418d/src/main/webapp/WEB-INF/jsp/vets/vetList__ScheduleVisit_withLink.jsp), which is a copy of the existing `vets/vetList.jsp` plus the implementation of the new _Availability_ column. This type of experience instrumentation, where the code bifurcates as early as possible, is referred to as _eager instrumentation_.

Updated for 0.9.3 on 8 October 2018.
