# MTA-STS Policy

This repository hosts the [MTA-STS](https://knowledge.workspace.google.com/admin/gmail/advanced/about-mta-sts-and-tls-reporting) policy for thewedge.org.

We're using GitHub Pages (see [documentation](https://docs.github.com/en/pages)) to make the policy file available, because it's free, supports custom domain names, and automatically obtains TLS certificates.

## What is MTA-STS?

In short, MTA-STS is a standard that tells computers sending messages to our domain that they must connect using Transport Layer Security (TLS), an encrypted protocol that prevents man-in-the-middle attacks. Without such a policy, some email senders may connect using a less secure protocol, allowing mail destined for our inboxes to be intercepted and potentially modified by an attacker.

To establish an MTA-STS policy for thewedge.org, we need two things:

* A policy file hosted at this exact URL: https://mta-sts.thewedge.org/.well-known/mta-sts.txt
  * The location is specified in [RFC 8461, Section 3.2](https://www.rfc-editor.org/info/rfc8461/#section-3.2). If our policy is hosted at any other location, mail servers won't find it.
  * HTTPS (which itself uses TLS underneath) is required. The file must be served with a valid TLS certificate, otherwise mail servers will refuse to fetch it.
* A DNS [TXT record](https://www.rfc-editor.org/info/rfc8461/#section-3.1) called `_mta-sts` containing a static protocol version string `v=STSv1;` and a unique policy instance ID.
  * This record must be updated with a new policy instance ID every time the policy file is updated, otherwise mail servers will continue using the previous policy until it ages out. Commonly the [Unix timestamp](https://www.unixtimestamp.com/) at the time of update is used for the instance ID, making a TXT record like: `v=STSv1;id=1780411575`.

The Google Admin console has a "compliance" section ([directions here](https://knowledge.workspace.google.com/admin/gmail/advanced/check-your-mta-sts-configuration#check_mta-sts_status_and_get_suggested_configurations)) that can suggest values for both the TXT record and the policy file, but updates have to be made manually in the DNS and this repo.

## Files in this repository

The `docs/` folder at the root of this repo is where GitHub Pages looks for content that it should host under `https://mta-sts.thewedge.org/`. Anything not under that directory will not show up. Within the folder, there are two special files:

* `docs/.nojekyll` is an empty file whose presence indicates to GitHub Pages that it should serve all files as-is, rather than using a program called Jekyll to generate the site. Weneed to serve a simple text file at a very specific path, so we want Jekyll disabled.
* `docs/CNAME` tells GitHub Pages that our site will be hosted at `https://mta-sts.thewedge.org`, rather than the default `https://thewedgempls.github.io/mta-sts/`.

The file at `docs/.well-known/mta-sts.txt` is the actual MTA-STS policy file, and the only file that'll end up being served by GitHub Pages.

## Making changes

GitHub Pages is configured to track the `main` branch of this repository for updates. Create a [pull request](https://github.blog/developer-skills/github/beginners-guide-to-github-creating-a-pull-request/) against `main` and merge it to kick off an update. It usually takes less than a minute for the update to finish.

You can load https://mta-sts.thewedge.org/.well-known/mta-sts.txt in your browser and refresh until you verify that the publicly-hosted file has changed.

After that, you'll need to go into the DNS configuration for thewedge.org and update the `_mta-sts` record with a new `id=` value, as mentioned above. Doing so will cause mail servers to re-fetch our policy and become aware of the new configuration.

### Turning off MTA-STS

If for some reason, you need to disable MTA-STS, you can't simply delete the policy file or the DNS TXT record (well, you *can* delete the TXT record, but the policy will stay in effect for longer than you probably intend). Follow Google's directions [here](https://knowledge.workspace.google.com/admin/gmail/advanced/turn-off-mta-sts) to edit the the policy file, combined with the directions above to apply those changes.
