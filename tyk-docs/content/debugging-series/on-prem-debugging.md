---
title: "On-Prem Debugging Guide"
date: 2023-01-06
tags: ["Debugging", "On-Prem Debugging", "Debugging Guide", "Debugging Tips" ]
description: "Debugging Tips and Tricks on how-to debug a Tyk Instance"
menu:
  main:
    parent: "Debugging Series"
aliases:
  - /debugging-series/on-prem-debugging/
---

This guide should help a user of Tyk's self managed offering in debugging common issues. A helpful way to go about this is by:

1. Isolating your components to see where the error is coming
2. Enabling debug logs to ensure you get all the information you need

## Gateway `/hello` endpoint

Querying the gateway's `/hello` health endpoint is the quickest way to determine the status of your Tyk instance. You can find more information in our docs about the [Gateway Liveness health check]({{< ref "planning-for-production/ensure-high-availability/health-check" >}}).

This endpoint is important as it allows the user to isolate the problem's origin. At a glance, the `/hello` endpoint reports the Gateways connectivity to Redis, and the control plane components eg. Dashboard, MDCB, Tyk Cloud. 

```json
{
    "status":"pass",
    "version":"v4.0.10",
    "description":"Tyk GW",
    "details":{
        "dashboard":{
            "status":"pass",
            "componentType":"system",
            "time":"2023-01-13T14:45:00Z"
            },
        "redis":{
            "status":"pass",
            "componentType":"datastore",
            "time":"2023-01-13T14:45:00Z"
            }
        },
        "rpc": {
            "status": "pass",
            "componentType": "system",
            "time":"2023-01-13T14:45:00Z"
        }
}
```

If the Dashboard or RPC connectivity fails, the Gateway will still function based on the last received configurations from those components respectively. If Redis fails, Gateway will not be able to perform since it is a hard dependency.

## Debug Logs

Having Debug mode turned on is helpful when debugging because it throws more descriptive error messages. For example, here are the different outputs you receive when calling an Open Keyless API with `info` and `debug` log level modes.

Here is the output when using `info` as the log level:

```
tyk-pump       | time="Jan 24 14:39:19" level=info msg="Purged 1 records..." prefix=mongo-pump
tyk-pump       | time="Jan 24 14:39:19" level=info msg="Purged 1 records..." prefix=mongo-pump-selective
tyk-mongo      | 2023-01-24T14:39:19.228+0000 I  NETWORK  [listener] connection accepted from 172.20.0.2:51028 #19 (19 connections now open)
tyk-pump       | time="Jan 24 14:39:19" level=info msg="Completed upserting" collection="tyk_analytics_aggregates" prefix=mongo-pump-aggregate
tyk-pump       | time="Jan 24 14:39:19" level=info msg="Purged 1 records..." prefix=mongo-pump-aggregate
```

Here is a more detailed output of the same call when using `debug` as the log level:

```
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg="Started proxy"
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg="Stripping proxy listen path: /api1/"
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg="Upstream path is: /get"
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg=Started api_id=63666619de884d0563ee3ccc67d57929 api_name=api1 mw=ReverseProxy org_id=63ca963f6888c7000191890e ts=1674570739659369736
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg="Upstream request URL: /get" api_id=63666619de884d0563ee3ccc67d57929 api_name=api1 mw=ReverseProxy org_id=63ca963f6888c7000191890e
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg="Outbound request URL: http://httpbin.org/get" api_id=63666619de884d0563ee3ccc67d57929 api_name=api1 mw=ReverseProxy org_id=63ca963f6888c7000191890e
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg="Creating new transport" api_id=63666619de884d0563ee3ccc67d57929 api_name=api1 mw=ReverseProxy org_id=63ca963f6888c7000191890e
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg="Out request url: http://httpbin.org/get" api_id=63666619de884d0563ee3ccc67d57929 api_name=api1 mw=ReverseProxy org_id=63ca963f6888c7000191890e
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg="Request is not cacheable" mw=ResponseCacheMiddleware
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg=Finished api_id=63666619de884d0563ee3ccc67d57929 api_name=api1 mw=ReverseProxy ns=316559477 org_id=63ca963f6888c7000191890e
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg="Upstream request took (ms): 316.639871"
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg="Checking: 63ca963f6888c7000191890e" api_id=63666619de884d0563ee3ccc67d57929 api_name=api1 org_id=63ca963f6888c7000191890e
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg="no cached entry found, returning 7 days" api_id=63666619de884d0563ee3ccc67d57929 api_name=api1 org_id=63ca963f6888c7000191890e
tyk-gateway    | time="Jan 24 14:32:19" level=debug msg="Done proxy"
tyk-pump       | time="Jan 24 14:32:20" level=info msg="Purged 0 records..." prefix=mongo-pump-aggregate
tyk-pump       | time="Jan 24 14:32:20" level=info msg="Purged 1 records..." prefix=mongo-pump-selective
tyk-pump       | time="Jan 24 14:32:20" level=info msg="Completed purging the records" collection="tyk_analytics" number of records=1 prefix=mongo-pump
tyk-pump       | time="Jan 24 14:32:20" level=info msg="Purged 1 records..." prefix=mongo-pump
tyk-mongo      | 2023-01-24T14:32:20.398+0000 I  NETWORK  [listener] connection accepted from 172.20.0.3:54712 #19 (19 connections now open)
tyk-pump       | time="Jan 24 14:32:20" level=info msg="Completed upserting" collection="tyk_analytics_aggregates" prefix=mongo-pump-aggregate
tyk-pump       | time="Jan 24 14:32:20" level=info msg="Purged 1 records..." prefix=mongo-pump-aggregate

```

As shown above, the `debug` log level mode throws more information which will help during your debugging stage, i.e when the API call was started, when it was finished, how long it took for the call to finish, the endpoint that was called, the upstream that was called, the organization that the API belongs to, and more.

### Gateway Debug Settings

If you’re using a `*.conf` for your configuration parameters:

```
"log_level": "debug"
```

If you’re using a `*.env` for your configuration:

```
TYK_GW_LOGLEVEL=debug
```

If you're using Tyk Helm Charts. Add the following items to your `values.yaml`:

```
extraEnvs:
  - name: TYK_LOGLEVEL
    value: debug
```

### Dashboard Debug Settings

If you’re using a `*.conf` for your configuration parameters:

```
"log_level": "debug"
```

If you’re using environment variables for your configuration:

```
TYK_DB_LOGLEVEL=debug
```

If you're using Tyk Helm Charts. Add the following items to your `values.yaml`:

```
extraEnvs:
  - name: TYK_LOGLEVEL
    value: debug
```

You can find the full [log levels]({{< ref "log-data" >}}) in our documentation.

## Versions/ Replicate

Please see the latest version releases [here](https://github.com/TykTechnologies/tyk/releases).

We recommend using the latest version of Tyk always. Tyk is backwards compatible, upgrading to newer versions won't activate any features or change behaviour of your existing environment

## Dashboard

It's essential to note that our Dashboard is built on top of our OSS, which means that the Dashboard uses the Dashboard API to retrieve its data. This means you can use the developer tools on your browser to access the API and its information. From here, you'll be able to see whether it's the Dashboard UI that's causing the error or if it's the API itself.

In many cases, you'll want to expose the same analytics you see in the Dashboard elsewhere or isolate if an error is on the [Gateway API]({{< ref "tyk-gateway-api" >}}) level or the [Dashboard API]({{< ref "tyk-dashboard-api" >}}) level.

By using the browser developer tools, you can see and replicate this information through API calls.

### Isolating

As mentioned above, errors can come from many parts of Tyk or your architecture, and as such, one of the critical things you'll pick up during your debugging phase is isolating these environments.

### Dashboard Level

When using the Dashboard, you'll run into a common theme where something doesn't work on the Dashboard but will work when using the Tyk Gateway API.

That's one of the first things you should do when debugging something on the Dashboard, i.e. does the same call work if you isolate the gateway away from the Dashboard? If it works with the gateway API only, then it's likely on the Dashboard level. This might be because the Dashboard needs some configuration parameters or environment variables.

### Gateway or API level

Are you making calls against your gateway or API, and it's not working? Try isolating the gateway from everything else. Often you'll see that the gateway or API aren't at fault and that it's something else; it can be the load balancer you have in your environment blocking the call from ever reaching it.

In the case of the API error-ing out, you can also isolate it by:

- Creating a generic Httpbin API and calling it
    - If this works, then the API configuration or the backend is at fault
- Changing the target URL of the API
    - The upstream API can be at fault
- Assuming your API has a plugin, take away the plugin and test the API
    - The error most likely exists in the plugin
- If the error exists in your plugin, try taking out certain parts of the code and testing it with minimal logic
    - This means that part of your code with integrated logic is incorrect
- Is the target URL the same in another one of your APIs?
    - The gateway sees the API as duplicated and changes the new target URL causing the gateway to error.

You will eventually hit the point of error by further isolating parts of your API.

## What do you do when you can’t fix the error?
You're probably not the first to encounter this error. Visit these relevant Tyk resources additional help or guidance:

1. [Tyk Community Forums](https://community.tyk.io/)
2. [Tyk FAQ Section]({{< ref "frequently-asked-questions" >}})
3. [Tyk Github Page](https://github.com/TykTechnologies)
4. Tyk Support (Paid SLA only)
