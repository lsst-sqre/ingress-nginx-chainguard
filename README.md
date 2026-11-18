# ingress-nginx (Rubin Observatory DM-SQuaRE fork)

## The lsst-sqre/ingress-nginx-chainguard fork

Unless you're Rubin Observatory's DM SQuaRE, you shouldn't be using this.
You should instead be using [the upstream source](https://github.com/chainguard-forks/ingress-nginx).

All we've done to it is to minimally rewrite the README.md, rip out a bunch of the CI that is not useful to us, and change the tags.

We are only intending to use this until we are able to migrate away from ingress-nginx to something using the Gateway API.

## NGINX source patches and ABI

We're not ingress-nginx maintainers, and we are not qualified to be.
If you found this repository while you were looking for some way to put a more modern NGINX into ingress-nginx-controller, you're welcome to try.
However, it's entirely on you to succeed or fail, and we accept no responsibility if something horrible happens to your Kubernetes deployment if you use this package.
We offer no support, and issues filed against it will most likely be cheerfully ignored.

## Rationale

This repository is designed to be a stopgap until Rubin Observatory can migrate to the K8s Gateway API.

We accept that [ingress-nginx is no longer maintainable](README-orig.md).

Nevertheless, we were unable to move away from ingress-nginx by the maintenance deadline and then the subsequent discovery of a major vulnerability.
Thus we've decided to rebuild the ingress-nginx controller with a version of NGINX that does not include the vulnerability.

## Changes from the parent repository

In addition to the obvious changes to the nginx base image and what repository it lives in, we've done a few other things:

 * We dropped almost all of the GitHub Actions, keeping only one that rebuilds the NGINX base container and the controller container.  We are not intending to maintain this package as a going concern.
 * We updated the GitHub Actions to their current (June 5, 2026) major versions and let them float within that major version.
 * We dropped 32-bit ARM support, leaving only amd64 and arm64 architectures.  Nothing in the Rubin environment that runs Kubernetes will ever need 32-bit ARM.
 * Updating NGINX is no longer done; we're just using whatever Chainguard uses in their build; fuzzing the patches got too difficult.

* Note that the controller name is now `ingress-nginx-controller`, not simply `controller`.  That's because `lsst-sqre` supplies other controllers (such as [Nublado](https://nublado.lsst.io)).  You will need to be aware of this when updating your helm charts.

### Update files

Increment the version numbers of the containers.
[TAG](TAG) holds the controller tag, and [images/nginx/TAG](images/nginx/TAG) holds the NGINX base container tag.
Keep the `-lsstsqre-chainguard` label that's appended to the tag.

Change [NGINX_BASE](NGINX_BASE) to pull the base image with the new base container tag.

Update [Changelog.md](Changelog.md) to note the NGINX upgrade.

Commit and push your changes, then open a pull request.

### Let the build run

GitHub Actions will do the base container build and the controller build.
It takes about two hours to rebuild the base container.
The controller is very quick after the base container is done.

### After the build

When everything is finished, `ghcr.io/lsst-sqre/nginx:NGINX_TAG` and `ghcr.io/lsst-sqre/ingress-nginx-controller:CONTROLLER_TAG` should both exist, and you can update [Phalanx](https://phalanx.lsst.io) to use the new controller container image.

### Git tidying

Merge your PR, and then create and push a git tag with the new value in [TAG](tag).

## How to update if you're not Rubin DM SQuaRE

You have to do all the steps above, but also you're going to need to change `ghcr.io/lsst-sqre` to whatever your container registry is.

[.github/workflows/build.yaml](.github/workflows/build.yaml) contains two instances of the `REGISTRY` env key with that value.  Also, if you're not using `ghcr.io`, you'll need to change the authentication information in three places, one for each `Login to GitHub Container Registry` step.

Then you'll need to change the `REGISTRY` definitions in [images/nginx/Makefile](images/nginx/Makefile) and [Makefile](Makefile).

The tags in [TAG](TAG), [images/nginx/TAG](images/nginx/TAG), and [NGINX_BASE](NGINX_BASE) should also probably get a label that is something other than `-lsstsqre-chainguard`; likewise for your [Changelog.md](Changelog.md) entry.

## Conclusion

This kicks the can down the road a little farther.
* First, it was August 13, 2026.
* There was another update on October 29, 2026.
* It is now November 18, 2026 and we're back for a third round.

Let's see how long it takes us to move away from ingress-nginx entirely.
