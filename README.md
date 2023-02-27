# fleet-infra

## Disclaimer

I don't provide support, feel free to open an issue but if you request help with
something I have in here, I will close it.

---

## Repo organization

Like most Flux repos, I have a `clusters` folder with a folder for every cluster
I've bootstrapped but at the moment, it's only one: `takeout`
(like takeout *containers*🥡, get it?).

Then it gets a little complicated because of CRDs and SOPS secrets.

`cluters/takeout` (has cluster tools like metallb, newrelic, generally things
that don't need other CRDs or secrets)

`custom-resources/takeout` (has prereq CRDs like cert-manager and storage that
depend on stuff in the `clusters/takeout` folder)

`customer-resource-consumers/takeout` (has most user apps that may need storage,
CRDs or secrets)

Here's the Kustomization hierarchy:

```
cluters/takeout
  \_⬆️ Depends on ⬆️ custom-resources/takeout
    \_⬆️ Depends on ⬆️ customer-resource-consumers/takeout
```

As you might assume, other clusters can be set up in these folders, but I only
have one right now. Flux takes about a minute to reconcile so there's room for
improvement but this organization should let me re-bootstrap without too many
race conditions.
