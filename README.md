# IP2Location.io Deno SDK — IP Geolocation, Domain WHOIS & Hosted Domain Lookup

The **IP2Location.io Deno SDK** lets Deno applications retrieve IPv4 and IPv6 geolocation, network, ASN, and available security intelligence through the [IP2Location.io API](https://www.ip2location.io/). It also provides domain WHOIS and hosted-domain lookup methods.

**Quick links:** [Get an API key](https://www.ip2location.io/pricing) · [Developer documentation](https://ip2location-io-deno.readthedocs.io/en/latest/) · [API reference](https://ip2location-io-deno.readthedocs.io/en/latest/code.html) · [GitHub repository](https://github.com/ip2location/ip2location-io-deno)

## Features

- **IPv4 and IPv6 geolocation:** Country, region, city, coordinates, ZIP/postal code, and time zone where available.
- **Network and ASN intelligence:** ASN, AS name, ISP, domain, and connection or usage information.
- **Proxy and security insights:** Available proxy indicators and fraud-related fields, depending on API plan.
- **Additional IP information:** Eligible country metadata, mobile network, weather, elevation, and advertising category fields.
- **Domain WHOIS:** Domain registration, registrar, nameserver, and available contact information.
- **Domain utilities:** Punycode conversion, domain extraction, and domain extension lookup.
- **Hosted-domain lookup:** Find domain names hosted on an IP address, with optional pagination.
- **Promise-based API:** Use familiar `async`/`await` syntax in Deno.

> **Note:** Available response fields, translations, quotas, and security indicators depend on your API subscription and the queried address.

## SDK Components

| Component | Method | Description |
|---|---|---|
| `Configuration` | `new Configuration(apiKey)` | Configure the API key. |
| `IPGeolocation` | `lookup(ip, language?)` | Retrieve geolocation and available network/security data. |
| `DomainWhois` | `lookup(domain)` | Retrieve available domain WHOIS information. |
| `DomainWhois` | `getPunycode(domain)` | Convert an internationalized domain to Punycode. |
| `DomainWhois` | `getNormalText(domain)` | Convert Punycode to Unicode. |
| `DomainWhois` | `getDomainName(url)` | Extract a domain from a URL. |
| `DomainWhois` | `getDomainExtension(domain)` | Extract a domain extension. |
| `HostedDomain` | `lookup(ip, page?)` | Retrieve domains hosted at an IPv4 or IPv6 address. |

For the complete return-field reference, see the [Deno SDK API documentation](https://ip2location-io-deno.readthedocs.io/en/latest/code.html).

## API Access and Query Limits

An [IP2Location.io API key](https://www.ip2location.io/pricing) is used in the SDK examples. A Free plan is available; higher plans provide additional data and query allowances. WHOIS and Hosted Domain API usage may have separate allowances.

IP2Location.io also offers limited **keyless REST API** access for geolocation requests; the examples in this README show the SDK's API-key configuration instead. Check the [pricing page](https://www.ip2location.io/pricing) for current limits, attribution requirements, and field availability.

## Requirements

- [Deno](https://deno.com/) installed.
- An [IP2Location.io API key](https://www.ip2location.io/pricing) for the SDK examples.
- Internet access to the API endpoints.
- Permission to access the network (`--allow-net`) and, for the examples below, to read the API key from the environment (`--allow-env=IP2LOCATION_API_KEY`).

## Installation

No npm installation is needed. Import the SDK directly from the module URL used in the [official Deno quickstart](https://ip2location-io-deno.readthedocs.io/en/latest/quickstart.html):

```javascript
import {
  Configuration,
  IPGeolocation,
  DomainWhois,
  HostedDomain,
} from "https://deno.land/x/ip2locationio@1.0.0/mod.ts";
```

Set your API key in the environment rather than embedding it in source code.

**Linux / macOS:**

```bash
export IP2LOCATION_API_KEY="YOUR_API_KEY"
```

**Windows PowerShell:**

```powershell
$env:IP2LOCATION_API_KEY = "YOUR_API_KEY"
```

## Quickstart: IP Geolocation Lookup

Create `main.ts`:

```typescript
import {
  Configuration,
  IPGeolocation,
} from "https://deno.land/x/ip2locationio@1.0.0/mod.ts";

const apiKey = Deno.env.get("IP2LOCATION_API_KEY");
if (!apiKey) throw new Error("Set IP2LOCATION_API_KEY first.");

const geo = new IPGeolocation(new Configuration(apiKey));

try {
  const result = await geo.lookup("8.8.8.8");
  console.log(result);
} catch (error) {
  console.error("IP lookup failed:", error);
}
```

Run the example:

```bash
deno run --allow-net --allow-env=IP2LOCATION_API_KEY main.ts
```

The SDK returns a JavaScript object parsed from the JSON API response. Fields vary by plan and IP address.

## More Usage Examples

The following examples assume the same imports and `apiKey` variable shown above. Network lookups return Promises, so call them in an async context and handle errors appropriately.

### Request Translated Location Names

Translation is available on eligible Plus and Security plans for supported location fields.

```typescript
const geo = new IPGeolocation(new Configuration(apiKey));
const result = await geo.lookup("8.8.8.8", "es");
console.log(result);
```

### Inspect ASN and Network Information

ASN identifies an Autonomous System, the network or group of networks operating under a common routing policy. ASN and AS-name data help with network troubleshooting, traffic analytics, and infrastructure identification.

```typescript
const geo = new IPGeolocation(new Configuration(apiKey));
const result = await geo.lookup("8.8.8.8");
console.log({
  asn: result.asn,
  asName: result.as,
  isp: result.isp,
  asInfo: result.as_info,
});
```

Additional AS metadata, such as domain, usage type, and CIDR, may be returned on eligible plans. For offline lookups, see [IP2Location ASN LITE](https://www.ip2location.com/database/lite).

### Inspect Available Proxy and Security Data

```typescript
const geo = new IPGeolocation(new Configuration(apiKey));
const result = await geo.lookup("8.8.8.8");
console.log({ isProxy: result.is_proxy, fraudScore: result.fraud_score, proxy: result.proxy });
```

These fields are plan-dependent and might be absent. Do not rely on a single IP signal to make definitive security decisions.

### Look Up Domain WHOIS Information

```typescript
const whois = new DomainWhois(new Configuration(apiKey));
const result = await whois.lookup("example.com");
console.log(result);
```

WHOIS output depends on the domain and available registration data; contact information may be redacted. For dedicated WHOIS integrations, see [IP2WHOIS](https://www.ip2whois.com/).

### Use Domain and Punycode Utilities

These helper functions work without an API request.

```typescript
const whois = new DomainWhois(new Configuration(apiKey));

console.log(whois.getPunycode("täst.de"));
console.log(whois.getNormalText("xn--tst-qla.de"));
console.log(whois.getDomainName("https://www.example.com/page"));
console.log(whois.getDomainExtension("example.com"));
```

### Find Domains Hosted on an IP Address

```typescript
const hosted = new HostedDomain(new Configuration(apiKey));
const firstPage = await hosted.lookup("8.8.8.8");
console.log(firstPage);

// Optional: request a specific page.
const secondPage = await hosted.lookup("8.8.8.8", 2);
console.log(secondPage);
```

## Deno-Specific Notes and Troubleshooting

| Issue | Suggested resolution |
|---|---|
| Permission denied for network access | Run with `--allow-net`. |
| Permission denied for environment access | Add `--allow-env=IP2LOCATION_API_KEY`. |
| API key missing or rejected | Confirm the environment variable and API subscription. |
| Module import fails | Check the import URL and your connection to `deno.land`. |
| HTTP request fails | Check connectivity, the API response, and quota status. |
| Fields are missing | Check plan entitlements and whether those fields apply to the requested IP. |
| WHOIS data is incomplete | Registration details may be unavailable or privacy-redacted. |

The SDK uses Deno's Node compatibility imports (`node:https` and `node:net`) and an npm Punycode dependency internally. The imported version in the examples is the one published in the project's official Deno quickstart.

## Common Use Cases

- **Localization:** Adapt content using approximate visitor location and language information.
- **Traffic analytics:** Enrich logs with country, ISP, or ASN data.
- **Network operations:** Research network operators and routing-related context.
- **Security workflows:** Combine available proxy and risk signals with other evidence.
- **Domain research:** Retrieve WHOIS details and hosted-domain lists.

## Related IP2Location Products

| Product | Purpose |
|---|---|
| [IP2Location.io](https://www.ip2location.io/) | Hosted IP geolocation, ASN, and network intelligence APIs. |
| [IP2Location LITE](https://www.ip2location.com/lite) | Free downloadable IP geolocation and ASN databases for offline use. |
| [IP2Location Database](https://www.ip2location.com/database/ip2location) | Commercial offline IP geolocation databases. |
| [IP2Proxy](https://www.ip2location.com/database/proxy) | IP proxy, VPN, TOR and anonymizer intelligence. |
| [IP2WHOIS](https://www.ip2whois.com/) | Domain WHOIS lookup services. |

## Accuracy and Responsible Use

IP geolocation is **not GPS**. It estimates the location associated with an IP address and may not represent the device's precise physical position. VPNs, mobile networks, corporate gateways, and shared infrastructure can affect the results. Use geolocation and proxy intelligence as supporting signals, not definitive evidence of identity, location, or malicious activity.

## Documentation and Support

- [Deno SDK quickstart](https://ip2location-io-deno.readthedocs.io/en/latest/quickstart.html)
- [Deno SDK API reference](https://ip2location-io-deno.readthedocs.io/en/latest/code.html)
- [IP2Location.io API documentation](https://www.ip2location.io/ip2location-documentation)
- [IP2Location.io API and plans](https://www.ip2location.io/pricing)
- [GitHub issues](https://github.com/ip2location/ip2location-io-deno/issues)

## License

Licensed under the **MIT License**. See [LICENSE](./LICENSE).
