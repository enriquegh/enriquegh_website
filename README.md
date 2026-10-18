# Enrique's website

A small, static Astro site. No client-side framework, external fonts, analytics,
or contact-form service. Edit the page and its styles in `src/pages/index.astro`.
The expandable sections use native HTML and work without JavaScript.

## Development

Use Node.js 24 LTS (minimum 22.22.0).

```sh
npm ci
npm run dev
```

```sh
npm run check
npm run build
npm run preview
```

The deployable site is generated in `dist/`. Files in `public/` are copied as-is.
The previous site's `assets/`, `images/`, and `enriqueghico/` directories are
retained as source material but are not included in the new build.

## GitHub Actions / S3

`.github/workflows/deploy.yml` replaces Travis. Pull requests to `main` run the
type check and production build. Pushes to `main` build and then deploy that
exact artifact to the existing `enriquegh.com` S3 bucket in `us-west-2`.
Manual runs deploy only when run against `main`.

Before the first deployment:

1. Create a GitHub environment named `production`. Add a required reviewer if
   deployments should wait for approval.
2. Configure an AWS IAM role for GitHub's OIDC provider, with audience
   `sts.amazonaws.com` and subject
   `repo:enriquegh/enriquegh_website:environment:production`. Since an environment
   subject replaces the branch subject, restrict that GitHub environment's
   deployment branches to `main`.
3. Grant the role `s3:ListBucket` on `arn:aws:s3:::enriquegh.com` and
   `s3:PutObject` on `arn:aws:s3:::enriquegh.com/*`.
4. Set the GitHub environment variable `AWS_DEPLOY_ROLE_ARN` to that role's ARN.

No long-lived AWS access keys are needed. The workflow does not change bucket
policies, object ACLs, DNS, or CDN configuration. It deliberately does not use
`--delete`: existing objects such as old image and résumé URLs remain intact.
New uploads use a five-minute cache lifetime. Any existing CDN configuration
may have its own cache policy; no CloudFront distribution is assumed here.

Repository secrets could not be inspected with the available GitHub token, so
AWS authentication and a live deployment must be verified after setup. Do not
remove old Travis AWS credentials until their other uses have been checked.
