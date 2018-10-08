![Variant Logo](http://www.getvariant.com/wp-content/uploads/2016/07/VariantLogoSquare-100.png)

# Variant Experiment Server Demo Application
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

__The feature toggle [`VetsHourlyRateFeature`](https://github.com/getvariant/variant-java-demo/blob/9affd4cc3992e8adf109a79532a1de75764ea38f/petclinic.schema#L44-L74)__ exposes an early release of a the new feature on the `Veterenarians` page, which adds the _Hourly Rate_ column to the table.

__The experiment [`ScheduleVisitTest`](https://github.com/getvariant/variant-java-demo/blob/9affd4cc3992e8adf109a79532a1de75764ea38f/petclinic.schema#L80-L142)__ validates another new featute, designed to improve new appointment bookings by displaying new _Availability_ column on the same same `Veterenarians` page. 

Since the  `Veterenarians` page is shared by bothvariations, it can have 4 variants:

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

Hybrid state variants are optional in Variant: unless explicitely configured in the schema, concurrent variations are treated as _disjoint_, i.e. Variant server will not target any sessions to variant experiences in both variations. However, in this demo, the more complex _conjoint_ concurrency model is demonstrated. It supports hybrid state vairants, when both variations are targeted to the variant experience. This is explained in detain in a subsequent section.

When you first navigate to the `Veterinarians` page, Variant server targets your session randomly in both variations. This targeting is _durable_, so reloading the page won't change it. If you want to make Variant to re-target, get a new private browser window. (Note that some browsers share cookies between private windows, so be sure that there are no other private windows open.) You may also vary the probability weights ([here](https://github.com/getvariant/variant-java-demo/blob/ad4d55d1d7017c64e4e9c8248f7700eff2c32b61/petclinic.schema#L54) and [here](https://github.com/getvariant/variant-java-demo/blob/ad4d55d1d7017c64e4e9c8248f7700eff2c32b61/petclinic.schema#L91)) by editing the `petclinic.schema` file in the server's `schemata` directory.

If your session happened to be targeted to the variant experience in the `ScheduleVisitTest` and you see the _Schedule visit_ link, clicking it will navigatete to the experimental version of the `New Visit`page, containing the vet's name:

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

The `STATE-VISIT` event is automatically triggered by Variant each time a user session visits an instrumented Web page. Note, that there will a delay between the time your session lands on an instrumented page and the event is flushed to the log. This is due to the asynchronous nature of Variant's event writer. You can reduce the lag by changing the value of the `event.writer.max.delay` configuration paramenter in the `conf/variant.conf` file. 

You can also [configure a different trace event flusher](http://getvariant.com/resources/docs/0-9/experience-server/reference/#section-4.3) to utilize a persistence mechanism of your choice. 

## 4. Discussion

Any online experiment starts with the implementation of variant experiences that will be compared to the existing code path. To accomplish this, we did the following:

1. Created controller mappings in the class <a href="https://github.com/getvariant/variant-java-servlet-adapter/blob/master/servlet-adapter-demo/src/main/java/org/springframework/samples/petclinic/web/OwnerController.java#L79-L117" target="_blank">OwnerController.java</a> for the new resource paths `/owners/new/variant/newOwnerTest.tosCheckbox` and `/owners/new/variant/newOwnerTest.tosAndMailCheckbox` — the entry points into the new experiences. Whenever Variant targets a session for a non-control variant of the `newOwner` page, it will forward the current HTTP request to that path. Otherwise, the request will proceed to the control page at the originally requested path `/owners/new/`.

2. Created two new JSP pages [`createOrUpdateOwnerForm__newOwnerTest.tosCheckbox.jsp`](https://github.com/getvariant/variant-java-servlet-adapter/blob/master/servlet-adapter-demo/src/main/webapp/WEB-INF/jsp/owners/createOrUpdateOwnerForm__newOwnerTest.tosCheckbox.jsp) and  [`createOrUpdateOwnerForm__newOwnerTest.tosAndMailCheckbox.jsp`](https://github.com/getvariant/variant-java-servlet-adapter/blob/master/servlet-adapter-demo/src/main/webapp/WEB-INF/jsp/owners/createOrUpdateOwnerForm__newOwnerTest.tosAndMailCheckbox.jsp), implementing the two new variants of the new owner page.

3. Created the experiment schema. 
```
//
// Variant Java client + Servlet adapter demo application.
// Demonstrates instrumentation of a basic Variant experiment.
// See https://github.com/getvariant/variant-java-servlet-adapter/tree/master/servlet-adapter-demo
// for details.
//
// Copyright © 2015-2017 Variant, Inc. All Rights Reserved.

{
   'meta':{
      'name':'petclinic',
      'comment':'Experiment schema for the Pet Clinic demo application'
   },
   'states':[                                                
     { 
       // The New Owner page used to add owner at /petclinic/owners/new 
       'name':'newOwner',
       'parameters': [
            {
               'name':'path',
               'value':'/petclinic/owners/new'
            }
        ]
     },                                                    
     {  
       // The Owner Detail page. Note that owner ID is in the path,
        // so we have to use regular expression to match.
       'name':'ownerDetail',
       'parameters': [
            {
               'name':'path',
               'value':'/petclinic/owners/~\\d+/'
            }
        ]
     }                                                     
   ],                                                        
   'tests':[                                                 
      {                                                      
         'name':'NewOwnerTest',
         'isOn': true,                                     
         'experiences':[                                     
            {                                                
               'name':'outOfTheBox',                                   
               'weight':1,                                  
               'isControl':true                              
            },                                               
            {                                                
               'name':'tosCheckbox',                                   
               'weight':1                                   
            },                                               
            {                                                
               'name':'tosAndMailCheckbox',                                   
               'weight':1                                   
            }                                               
         ],                                                  
         'onStates':[                                         
            {                                                
               'stateRef':'newOwner',                    
               'variants':[                                  
                  {                                          
                     'experienceRef': 'tosCheckbox',
                     'parameters': [
                        {
                           'name':'path',
                           'value':'/owners/new/variant/newOwnerTest.tosCheckbox'
                        }
                     ]
                  },                                         
                  {                                          
                     'experienceRef': 'tosAndMailCheckbox',                   
                     'parameters': [
                        {
                           'name':'path',
                           'value':'/owners/new/variant/newOwnerTest.tosAndMailCheckbox'
                        }
                     ]
                  }                                          
               ]                                             
            },
            {                                                
               'stateRef':'ownerDetail',                            
               'isNonvariant': true
            }                                                
         ],
         'hooks':[
            {
               // Disqualifies all Safari traffic.
               'name':'SafariDisqualifier',
               'class':'com.variant.server.ext.demo.SafariDisqualHook'
            },
            {
               // Assigns all Chrome traffic to the control experience
               'name':'ChromeTargeter',
               'class':'com.variant.server.ext.demo.ChromeTargetingHook'
            }
         ]
      }                                                     
   ]                                                         
}                         
      

```
Note, that Variant server comes out-of-the-boxwith this schema already in the `schemata` directory.

The two states (lines 14-30) correspond to the two consecutive pages in the experiment: <span class="variant-code">newOwner</span> and <span class="variant-code">ownerDetail</span>. The sole experiment <span class="variant-code">NewOwnerTest</span> has three experiences (lines 35-49) with equal weights, i.e. roughly equal number of users sessions will be targeted to each of the experiences. The test is instrumented on both pages, although the <span class="variant-code">ownerDetail</span> page is defined as non-variant (lines 68-71), which means that visitors will see the same page regardless of the targeted experience. The <span class="variant-code">newOwner</span> page, however, has two variants: one for each non-control experience (lines 53-66).

4. Created [`PetclinicVariantFilter`](https://github.com/getvariant/variant-java-servlet-adapter/blob/master/servlet-adapter-demo/src/main/java/com/variant/client/servlet/demo/PetclinicVariantFilter.java) and configured it in the Petclinic applciation's [`web.xml`](https://github.com/getvariant/variant-java-servlet-adapter/blob/master/servlet-adapter-demo/src/main/webapp/WEB-INF/web.xml#L124-L137) file. In general, servlet filter is the instrumentation mechanism behind the servlet adapter. Here, we extend the base [`VariantFilter`](https://github.com/getvariant/variant-java-servlet-adapter/blob/master/servlet-adapter/src/main/java/com/variant/client/servlet/VariantFilter.java) class with additional semantics: whenever the base `VariantFilter` obtains a Variant session, the User-Agent header from the incoming request is saved as a session attribute. This will be used by the server side user hooks in order to disqualify or target user sessions based on what Web browser they are coming from.

5. Created [FirefoxDisqualifier](https://github.com/getvariant/variant-server-extapi/blob/master/server-extapi-demo/src/main/java/com/variant/server/ext/demo/FirefoxDisqualHook.java) and [ChromeTargeter](https://github.com/getvariant/variant-server-extapi/blob/master/server-extapi-demo/src/main/java/com/variant/server/ext/demo/ChromeTargetingHook.java) user hooks. Out-of-the-box, Variant server comes with the `/ext/server-extensions-demo-0.7.1.jar` JAR file, containing class files for these hooks. See [Variant Experiment Server Reference](http://www.getvariant.com/docs/0-7/experiment-server/reference/#section-4) for details on how to develop for the server extensions API. 

6. Instrumented the submit button on the `newOwner` page to send a custom `CLICK` event to the server when the button is pressed by editing the [`staticFiles.jsp`](https://github.com/getvariant/variant-java-servlet-adapter/blob/master/servlet-adapter-demo/src/main/webapp/WEB-INF/jsp/fragments/staticFiles.jsp#L32-L68) file.




Updated on 19 July 2017.
