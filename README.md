# Portfolio Site — AWS Deployment

My personal portfolio, deployed on AWS with a free custom domain and HTTPS.

**Live site:** https://chaitanyat.is-a.dev

## Architecture

- **Hosting:** Amazon S3 (static website)
- **CDN + HTTPS:** Amazon CloudFront
- **SSL Certificate:** AWS Certificate Manager (ACM), DNS-validated
- **DNS / Domain:** [is-a.dev](https://is-a.dev) — free subdomain registry, `chaitanyat.is-a.dev`
- **Cost:** $0/month (within AWS Free Tier), monitored with a zero-spend AWS Budget alarm


## Why this setup

Rather than a simple S3 public-bucket deployment, I used CloudFront in front of a **private** S3 bucket (via Origin Access Control) so the bucket itself isn't publicly exposed — CloudFront is the only path in. This also unlocks free HTTPS via ACM, which a bare S3 website endpoint doesn't support.

## Challenges & Troubleshooting

**1. CloudFront 403 on custom domain**
After pointing my is-a.dev CNAME at the CloudFront distribution, requests to the custom domain returned a `403: The request could not be satisfied`. Root cause: CloudFront only serves requests whose `Host` header matches its default `*.cloudfront.net` domain or a registered **Alternate Domain Name (CNAME)** — it doesn't automatically accept traffic just because DNS routes to it. Fixed by explicitly adding the custom domain as an Alternate Domain Name on the distribution and attaching a matching ACM certificate.

**2. ACM certificate requires DNS validation via a domain I don't fully control**
Since is-a.dev is a third-party free registry (not Route 53), ACM's DNS validation CNAME had to be added through a second pull request to the is-a.dev registry rather than directly in a DNS console. This meant learning their domain-file schema for nested/underscore-prefixed subdomains (e.g. `_<hash>.chaitanyat.json`) to represent the validation record correctly.

**3. S3 origin vs. S3 website endpoint trade-off**
CloudFront's console nudges you toward using the S3 *website endpoint* as the origin when it detects static hosting is enabled. I intentionally avoided this — the website endpoint only supports HTTP (no HTTPS to origin) and requires the bucket to stay public. Using the S3 REST API origin with Origin Access Control instead keeps the bucket private and supports HTTPS end-to-end, at the cost of manually setting a **Default Root Object** (`index.html`), which the website endpoint would have handled automatically.

**4. Free-tier cost visibility**
To avoid any surprise billing on a personal AWS account, I set up a zero-spend AWS Budget with email alerts — it fires the moment any charge above $0.01 is recorded, rather than waiting to cross a dollar threshold.
