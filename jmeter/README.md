## General tips on .jmx config file

The [sample.jmx](./sample.jmx) includes some modules to configure the HTTP request, headers and the body that Azure Cognitive Search is expecting. It also includes subsections to configure the query distribution (ie 10 concurrent users per second during 1 minute), a section to define which search terms will be sent (to avoid distortion in latencies thanks to cache) that read an input CSV. For more details and examples: [JMeter official doc](https://jmeter.apache.org/usermanual/component_reference.html).

If you struggle adding new modules to the .jmx (the syntax can be quite tricky) I would suggest to use JMeter's UI and save the config to a temporary jmx file, analyze the new module and embed it in your jmx config file. There is also an online editor to simplify drafting the test strategy: https://jmeter-plugins.org/editor/

### ThreadGroup 

This section defines the test strategy, ie "ThreadGroup.on_sample_error" if the test should stop once it encounters an error. "num_threads" is the number of calls that the service will receive along the period ("RampUp") 
```xml
        <stringProp name="ThreadGroup.num_threads">180</stringProp>
        <stringProp name="ThreadGroup.ramp_time">60</stringProp>
        <boolProp name="ThreadGroup.scheduler">true</boolProp>
        <stringProp name="ThreadGroup.duration">60</stringProp>
        <stringProp name="ThreadGroup.delay"></stringProp>
        <boolProp name="ThreadGroup.same_user_on_next_iteration">false</boolProp>

```


### HTTPSamplerProxy 

This section includes the parameters and the body of your REST API call, must adhere to [the expected Azure Cognitive Search syntax](https://docs.microsoft.com/en-us/azure/search/query-lucene-syntax). You can set the search instance, the index name, the api-version and the "Argument.value" itself includes the search body (in this case a  term from a defined variable, that reads from a CSV list)

```xml
          <elementProp name="HTTPsampler.Arguments" elementType="Arguments">
            <collectionProp name="Arguments.arguments">
              <elementProp name="" elementType="HTTPArgument">
                <boolProp name="HTTPArgument.always_encode">false</boolProp>
                <stringProp name="Argument.value">{  &#xd;
     &quot;search&quot;: &quot;${searchterm}&quot;,  &#xd;
     &quot;skip&quot;:0,&#xd;
     &quot;top&quot;: 5,&#xd;
     &quot;queryType&quot;: &quot;full&quot;&#xd;
}</stringProp>
                <stringProp name="Argument.metadata">=</stringProp>
              </elementProp>
            </collectionProp>
          </elementProp>
          <stringProp name="HTTPSampler.domain">your_instance.search.windows.net</stringProp>
          <stringProp name="HTTPSampler.port">443</stringProp>
          <stringProp name="HTTPSampler.protocol">https</stringProp>
          <stringProp name="HTTPSampler.contentEncoding"></stringProp>
          <stringProp name="HTTPSampler.path">/indexes/your_index_name/docs/search?api-version=2020-06-30</stringProp>
```

The timeouts are optional, in this case set at 10 secs
```xml
          <stringProp name="HTTPSampler.connect_timeout">10000</stringProp>
          <stringProp name="HTTPSampler.response_timeout">10000</stringProp>
```

### HeaderManager 

This section includes the header values needed for the REST API to go through the service. API_KEY will be substituted by a Devops step for the real key

```xml
              <stringProp name="Header.name">api-key</stringProp>
              <stringProp name="Header.value">API_KEY</stringProp>
            </elementProp>
            <elementProp name="" elementType="Header">
              <stringProp name="Header.name">Content-Type</stringProp>
              <stringProp name="Header.value">application/json</stringProp>
```

### CSVDataSet 

The search engine has a cache, if you repeat the same query the latency seen in the results will not be realistic compared to scenario where your users query the system with diverse terms. This module defines the input list used to query starting from first line (if you need a random ordered term from the CSV use https://www.blazemeter.com/blog/introducing-the-random-csv-data-set-config-plugin-on-jmeter)

### Constant Throughput Timer 

Useful module to set a constant rate of requests per second. The main property is the requests per minute to maintain along the thread life, [more info here](https://www.blazemeter.com/blog/how-use-jmeters-throughput-constant-timer)

### RegexExtractor and BeanShellAssertion 

In this case they are used to analyze the responses to our queries and extract one particular field: elapsed-time that will be dumped into the results.jtl. Elapsed-time accounts for the internal latency of Azure Cognitive Search engine, it is useful to understand how does the full latency break down into networking, transmission (RTT) and search engine time

## Examples

### Example 1: Simple test scenario, 3 requests per second over a minute
[`sample.jmx`](./sample.jmx)


### Example 2: Step growth scenario using "Concurrency Thread Group", ramp up towards 100 concurrent requests along 5 steps
[`sample_steps.jmx`](./sample_steps.jmx)

More info on [Concurrency Thread Group Plugin](https://jmeter-plugins.org/wiki/ConcurrencyThreadGroup/)
