# fleet-infra

## Disclaimer

I don't provide support, feel free to open an issue but if you request help with
something I have in here, I will close it.

---

## Repo organization

Like most Flux repos, I have a `clusters` folder with a folder for every cluster
I've bootstrapped. `corpus` is the main cluster, 3 control panel nodes, 3 workers. `workbench` is a single node cluster on a micro form factory system. It is the k8s load balancer for corpus, runs the unifi controller and home assistant, etc. The intention is to make it as stand-alone as possible so if there are cluster issues or issues with my router (which previously was the k8s load balancer), it would remain up or be able to be restarted without relying on anything else.

The Customization organization is pretty straightforward, a folder for Flux-related things and then another folder for apps. I've taken the time to declare dependencies for everything so re-bootstrapping is hands-off.

