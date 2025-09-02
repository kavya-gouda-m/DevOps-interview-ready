### Azure Application Gateway started dropping requests randomly under load. Health probes are green. What next?

- The Application Gateway itself is hitting a limit.
- The backend servers are overwhelmed and can't process requests fast enough.
- A network component between them is failing under stress.

Check the Metrics:
in Azure Monitor for your Application Gateway and look at these key metrics.
- Failed Requests: Is this metric spiking when the drops occur? This is your primary indicator.
- Current Connections: Is this number hitting a plateau? You might be reaching a connection limit on the gateway or the backend
- Healthy Host Count & Unhealthy Host Count: You said probes are green, but are they flapping? Look for rapid changes between healthy and unhealthy. A server that's overloaded might fail a probe for a few seconds and then recover, which can look "random."
- Response Status: Check the distribution. Are you seeing a lot of 5xx status codes? Specifically, a 502 Bad Gateway or 504 Gateway Timeout points directly to backend issues.
- Application Gateway total time and Backend connect time: If these metrics are high, it indicates slow responses from the backend servers.

You should also query the Application Gateway Access Logs ($ApplicationGatewayAccessLog_CL) in a Log Analytics Workspace. Look for requests with a httpStatus of 502, 503, or 504. Pay close attention to the timeTaken field for those requests

Investigate the Application Gateway

If the metrics point to the gateway itself being the bottleneck, check these things.
-  SKU and Instance Count: Are you on V1 or V2 SKU? This is critical.
    - V1 SKU: This uses Capacity Units and requires manual scaling. If your load has increased, you may simply need to increase the instance count. You're likely hitting the CPU or network throughput limit of your deployed instances.
    - V2 SKU: This autoscales based on traffic. However, it's not instantaneous. Check the Scale Units metric. A high number of scale units means it's working hard. V2 can still have limits, so ensure your maximum instance count in the autoscale settings is high enough.
-  Request Timeout: Check your HTTP Settings. The default is 20 seconds. If your backend takes longer than this to respond under load, the Application Gateway will terminate the connection and return a 504 Gateway Timeout to the client. The backend might still be processing the request, but as far as the gateway is concerned, it failed.
-  WAF (Web Application Firewall): If you're using the WAF, check the firewall logs ($ApplicationGatewayFirewallLog_CL). Is it in Prevention mode? Under load, you might be triggering a complex rule (e.g., a regex) that is causing performance degradation or even false positives that block legitimate requests.

Scrutinize the Backend Pool

A green health probe often just pings a lightweight endpoint like /health which doesn't simulate real work.

- Backend Server Performance: Log into the backend VMs or check the App Service Plan metrics. Are you maxing out CPU, Memory, or Disk I/O during the periods of high load? A server at 99% CPU can still respond to a simple health probe, but it will struggle to handle a complex API request, causing it to queue up and eventually time out
- Application-Level Bottlenecks: The problem might be deeper in the application stack.
  - Thread Pool Exhaustion: Your web server (IIS, Nginx, Kestrel) might be running out of available worker threads to handle incoming requests.
  - Database Connections: Is your application waiting for connections from a maxed-out database connection pool?
  - Upstream API Calls: Is your application calling another service that is slow under load?
- Improve the Health Probe: The long-term fix here is to make your health probe more realistic. Instead of just hitting a static page, have it perform a quick, read-only check that mimics a real user action, like a quick database lookup. This gives a much more accurate picture of health.

Check the Network Path

- Network Security Groups (NSGs): It's unlikely that an NSG is causing random drops, but a misconfiguration is always possible. Double-check your rules to ensure nothing is being blocked.
- SNAT Port Exhaustion: This is a subtle but very common issue. If your backend servers need to make a lot of outbound calls to the internet (e.g., to external APIs), they can run out of available SNAT ports on the Azure Load Balancer. This will cause new outbound connections to fail, which could make your application appear unresponsive to the Application Gateway.
  - Diagnosis: Check the SNAT Connections metric on your Load Balancer.
  - Solution: The best practice is to attach a NAT Gateway to your backend subnet. This provides a dedicated pool of thousands of SNAT ports and effectively eliminates this problem.  
