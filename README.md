# RevIP

> The more silent you are, the more you can hear.

RevIP gathers public information associated with an IP address or CIDR range from a Linux terminal. It is a single-file Python tool with optional API-key integrations.

Use RevIP only on systems you own or have permission to assess. Active checks can be detected by the target and may trigger security controls.

## Features

Depending on enabled sources and provider availability, RevIP can collect:

- Registration details, allocation dates, and abuse contacts
- Reverse DNS and forward-confirmed DNS results
- Origin ASN, announced prefix, and neighboring prefixes
- RPKI route-origin validation status
- Geolocation, ISP, and timezone information
- Reverse-IP results and certificate-transparency hostnames
- Archived URLs and mail-authentication records
- Public blocklist results and a weighted VPN, proxy, or Tor assessment
- Certificate details, TLS fingerprints, and service responses from active checks
- Results from optional keyed services, including Shodan, VirusTotal, and AbuseIPDB

The final report may classify an address as `listed`, `tor exit`, `proxy / vpn`, `datacenter`, or `residential-ish`, based on the available results. These classifications are estimates, not definitive findings.

## Requirements

- Linux
- Python 3.9 or later
- `requests` and `PySocks`

On first run, RevIP attempts to install its dependencies. Use `--no-install` if you prefer to manage dependencies yourself.

## Installation

```bash
git clone https://github.com/bobo/revip.git
cd revip
chmod +x revip.py
```

## Data sources

Availability, quotas, and results depend on each provider.

| Source | Information |
|---|---|
| DNS | PTR records and forward-confirmed results |
| RDAP | Registry allocation, dates, and abuse contacts |
| BGP / ASN | Origin ASN, announced prefix, RIR, and AS name |
| ASN neighbors | Other prefixes announced by the origin |
| RPKI | ROA validity and invalid-route status |
| GEO | Country, region, city, coordinates, ISP, ASN, and timezone |
| Reverse IP | Domains resolving to the address |
| HackerTarget | Additional reverse-IP results |
| CT logs | Hostnames found in certificate-transparency records |
| Wayback | Archived URLs and paths |
| Mail auth | MX, SPF, and DMARC records for discovered domains |
| Robtex | Passive DNS history and hostnames |
| InternetDB | Known ports, hostnames, CPEs, and CVEs |
| Active checks | Service responses, banners, virtual hosts, and TLS details |
| DNSBL | Listings from five blocklists |
| VPN / proxy | Tor exit-list, datacenter-range, and weighted-score checks |
| Keyed APIs | Optional integrations such as Shodan, VirusTotal, AbuseIPDB, ipinfo, and ip-api Pro |

Private, loopback, and reserved addresses skip certain API sources. Other checks may still run.

## Routes and network behavior

RevIP supports direct connections, locally detected Tor SOCKS ports, and custom SOCKS or HTTP proxies. When a route is configured, RevIP applies it to supported requests and active checks.

DNS queries use DNS over HTTPS through Cloudflare or Google. Other provider requests use HTTPS, except the free ip-api tier, which uses plaintext HTTP.

A proxy or Tor route does not make active checks invisible to their targets. The target may still see and log the connecting address.

## API keys

Optional API keys can increase provider quotas or add data. RevIP stores keys in its configuration file as plain text with file permissions set to `0600`. Keys are not included in reports.

The configuration file is located at:

```text
$XDG_CONFIG_HOME/revip/config.toml
```

If `XDG_CONFIG_HOME` is not set, RevIP uses the system’s default XDG configuration directory.

| Provider | Quota listed in project notes | Additional information |
|---|---:|---|
| Shodan | 1 request per second | Host record, banners, and CVEs |
| VirusTotal | 4 requests per minute | Detection ratio, certificates, and related hosts |
| AbuseIPDB | 1,000 requests per day | Abuse reports and confidence |
| ipinfo | 50,000 requests per month | ASN, privacy, company, and abuse contact |
| ip-api Pro | Unlimited | SSL geolocation lookup |
| DNSRift | 15,000 requests per day | Higher reverse-IP quota |

The project notes list these limits for keyless sources: HackerTarget (50 per day), ip-api (45 per minute), and Robtex (10 per hour). Remaining quota is shown when a provider returns quota headers.

## Output and cache

RevIP can produce JSON output for use in scripts. In JSON mode, status messages are sent to stderr so stdout remains parseable.

Each source reports a status (`ok`, `error`, `skip`, or `stop`), data, and a note.

Responses are cached for 15 minutes by default. The cache can be cleared from the interface or disabled with `--no-cache`.

The cache directory is located at:

```text
$XDG_CACHE_HOME/revip/
```

## Limits and reliability

RevIP applies time limits to sources so a stalled provider does not keep a run open indefinitely. Optional sources may stop running after the configured scan budget is reached. Providers may also impose their own quotas or change their terms and availability.

## Security and privacy considerations

- The free ip-api tier uses plaintext HTTP. A network observer may see the address being looked up.
- Provider certificates are not pinned. A trusted local certificate authority could inspect traffic.
- Tor hides your address from providers, but not the Tor exit address from the target.
- Archived URLs may contain query parameters or tokens.
- Configuration files contain API keys in plain text. `0600` restricts access for other local users, but not root.
- RevIP identifies itself to providers with its user agent.
- Active checks can appear in target logs and may result in blocking.

## Responsible use

Scan only systems you own or are explicitly authorized to assess. Public availability of data or services does not imply permission to probe them. Follow provider terms and applicable laws.

## Provider terms

RevIP relies on public and free-tier sources. Providers may change their quotas, terms, or availability at any time.
