## DDoS mitigation with AWS WAF for geo-specific business

Russian 'Hacktivists' Are Causing Trouble Far Beyond Ukraine. Many organisations in Europe are being targeted with DDoS attacks. Banks, Parlements sites and universities are just a few of the organisations under these Russian 'Hacktivists' attacks.

But not all is bad; many of these targets can mitigate an attack with a simple AWS WAF rule.
We will focus on the targets in which most of their traffic comes from a particular subset of countries. 

For example, an Estonian news outlet like https://news.postimees.ee/, which focuses on Estonian news also written in Estonian, has most of its traffic coming from Estonia.

## The first rule
That comes to mind that could be found easily online is an AWS WAF rule that would restrict all traffic not coming from a specific country, in this case, Estonia.

```
{
  "Name": "OnlyEstonia",
  "Priority": 0,
  "Action": {
    "Block": {}
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "OnlyEstonia"
  },
  "Statement": {
    "GeoMatchStatement": {
      "CountryCodes": [
        "EE"
      ]
    }
  }
}
```

### The problem:
With the previous rule, protecting your services will block legitimate traffic outside Estonia, affecting business.

## The second rule
That comes to mind that would overcome the previous problem would be to rate limit requests by IP.

```
{
  "Name": "RateLimiter",
  "Priority": 0,
  "Action": {
    "Block": {}
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "RateLimiter"
  },
  "Statement": {
    "RateBasedStatement": {
      "Limit": "2000",
      "AggregateKeyType": "IP"
    }
  }
}
```

### The problem:
Due to proxies and gateways, you can have valid traffic coming in with the same IP that would look like an attack, or you could have attacking traffic and valid traffic with the same IP.

## The sweet spot 
is a mix of both rules. Given the example, the best thing is not to limit traffic from Estonia (or have a very high limit) and rate-limit (not block ) traffic from other countries.

```
{
  "Name": "Rate-limit-eveything-but-estonia",
  "Priority": 0,
  "Action": {
    "Block": {}
  },
  "VisibilityConfig": {
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "Rate-limit-eveything-but-estonia"
  },
  "Statement": {
    "RateBasedStatement": {
      "Limit": "100",
      "AggregateKeyType": "IP",
      "ScopeDownStatement": {
        "NotStatement": {
          "Statement": {
            "GeoMatchStatement": {
              "CountryCodes": [
                "EE"
              ]
            }
          }
        }
      }
    }
  }
}
```

## In Summary
Check your logs, measure regular traffic rates, and add this rule to make your enviroment safer.