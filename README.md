<div align="center">
<h1>dns_resolver</h1>

A recursive DNS resolver written in Python using raw UDP sockets. Resolves 
domains by walking the full delegation chain ,  Root servers → TLD nameservers 
→ Authoritative nameservers.

[![License](https://img.shields.io/github/license/so1icitx/dns_resolver)](LICENSE)
</div>

## How it works

Every query triggers a full recursive resolution from scratch:

1. A random root server is selected from the 13 IANA root server IPs
2. The TLD nameserver (e.g. `.com`) is extracted from the root response
3. The authoritative nameserver for the domain is resolved from the TLD response
4. The final query is sent to the authoritative nameserver and the answer is relayed back to the client

DNS packets are parsed manually using `struct.unpack` directly on the raw 
byte stream. The question section, authority records, and additional records 
are all walked by hand to extract nameserver IPs and glue records.

## Getting started

Requires Python 3.x. No external dependencies.

```bash
git clone https://github.com/so1icitx/dns-resolver.git
cd dns-resolver
```

The server binds to port `53` and may require root privileges:

```bash
sudo python3 main.py
```

Test with `nslookup`:

```bash
nslookup google.com 127.0.0.1
```

## Known limitations

- Glue record handling is fragile ,  if a nameserver's IP isn't in the 
  additional section of the response, resolution fails
- UDP only; no TCP fallback for responses over 512 bytes
- No caching ,  every query walks the full chain from root servers
- No support for reverse lookups (`arpa` queries are silently dropped)
- Single-threaded; one query at a time
