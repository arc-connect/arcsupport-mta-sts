# arcsupport-mta-sts

Hosts the **MTA-STS policy** for `arcsupport.ca` via GitHub Pages.

MTA-STS ([RFC 8461](https://datatracker.ietf.org/doc/html/rfc8461)) tells sending
mail servers to require TLS when delivering mail to our domain (Microsoft 365 /
Outlook). The policy file must be served over **valid HTTPS** at exactly:

```
https://mta-sts.arcsupport.ca/.well-known/mta-sts.txt
```

## How this works

- This repo serves that one file through **GitHub Pages**.
- `CNAME` binds the custom domain `mta-sts.arcsupport.ca` to this Pages site.
- TLS is provisioned automatically by GitHub Pages.
- The DNS lives **outside this repo** (see below) — currently in our existing
  DNS provider, with a planned migration to **AWS Route 53**. Moving the zone
  does not affect this repo; just recreate the same records in Route 53.

## The policy file

`.well-known/mta-sts.txt`:

```
version: STSv1
mode: testing
mx: *.mail.protection.outlook.com
max_age: 604800
```

`mode: testing` means receivers **report** TLS failures but still deliver mail.
Once validated, switch to `mode: enforce` to make TLS mandatory.

## Required DNS records (kept in our DNS provider, not here)

```
mta-sts.arcsupport.ca.     CNAME   arc-connect.github.io.
_mta-sts.arcsupport.ca.    TXT     "v=STSv1; id=2026062301"
```

Optional TLS reporting:

```
_smtp._tls.arcsupport.ca.  TXT     "v=TLSRPTv1; rua=mailto:postmaster@arcsupport.ca"
```

## ⚠️ When you edit the policy

Every time you change `mta-sts.txt`, **bump the `id`** in the `_mta-sts` TXT
record (e.g. to a new `YYYYMMDDnn` stamp). Receivers cache the policy by `id`;
if it doesn't change, they keep serving the old one.

## Verify

After DNS propagates, confirm:

```
curl -sI https://mta-sts.arcsupport.ca/.well-known/mta-sts.txt
```

Expect `HTTP/2 200` and `content-type: text/plain`. Then test with a tool such
as <https://aykevl.nl/apps/mta-sts/> or <https://www.hardenize.com/>.
