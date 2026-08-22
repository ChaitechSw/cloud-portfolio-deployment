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
