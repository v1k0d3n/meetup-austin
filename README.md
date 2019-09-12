# Meetup Austin: Smoother Sailing with Helm v3

Thanks for attending my talk at the GDG Cloud Austin Meetup on Sept 12th 2019!

It was a pleasure to meet all of you. I'd really like to thank Victor and Jason for inviting me to come visit your group, along with the folks at [Volusion](https://www.volusion.com/) for being such great hosts (and of course, staying a bit late).

Please feel free to look through the repository we covered during the talk. Here's a rundown of some of the things you will find in here:

- **Dockerfile:** which was used to communicate with Tiller, where we expect the XSS exploit to occur from
- **Demo Folder:** used to run through the demo and to collect the bootstrap secret from the `kube-system` namespace)
- **Escalation Chart:** used to elevate `ClusterRole` and `ClusterRoleBinding` (used to collect secrets from the Kubernetes API)
- **Sample Charts:** v2 and v3 minimalist versions of our Calico CNI, along with a `kubernetes-common` Library chart
- **`calicoctl.cfg`:** used to configure the `calicoctl` client via Kubernetes `kubeconfig`
- **Calico Policy:** (that was used to demonstrate blocking `44134` internally via labels `app=helm || name=tiller`)


# Execution of the Meetup Demonstration

To set up an environment for this demonstration, perform the following steps:
1. Use [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) to set up a Kubernetes cluster with fairly default settings. I would suggest that you leverage the [kubeconfig](https://github.com/kubernetes/kubeadm/issues/235#issuecomment-311928904) example, as the `kubeadm` authors suggested for this demonstration.
2. Install the [Calico CNI](https://docs.projectcalico.org/v3.8/getting-started/kubernetes/installation/calico#installing-with-the-kubernetes-api-datastore50-nodes-or-less) so you can implement Calico Network Policy as I showed during our Meetup.
3. Use the [Helm](https://helm.sh/) client to [initialize Tiller](https://helm.sh/docs/install/).
4. Run the included [Dockerfile](https://cloud.docker.com/u/v1k0d3n/repository/docker/v1k0d3n/helm-security) in the example cluster with the following command: `kubectl run -n default --quiet --rm --restart=Never -ti --image=v1k0d3n/helm-security exposed`
5. Once you are inside of the container, you can run the following commands in sequence: `demo01`, `demo03`, `demo03`, and so on...

The scripts will `printf` the commands that are being executed inside the container.

# Prevention

You can very quickly and easily implement a fix for this type of vulnerability by using [Project Calico](https://www.projectcalico.org/). If you've run through the demonstration above, now you can implement a security policy by doing the following:

1. Download the `calicoctl` client, following the [instructions](https://docs.projectcalico.org/v3.8/getting-started/calicoctl/install#installing-calicoctl-as-a-binary-on-a-single-host) from the Project Calico website. For our demonstration, I used the binary instructions in the provided link reference.
2. Configure the `calicoctl` binary to use the Kubernertes datastore (an example is provided in the `calicoctl.cfg`). Take note of where your `kubeconfig` file is located.
3. Reset the demo (delete the escalation chart, and make sure the `exposed` pod is removed).
4. Apply the policy via `calicoctl apply -f calico-policy.yaml`.
5. Run through the demo like you did previously.

Did you notice any changes? You'll notice that your Helm client will continue to work as it did before, but since port `44134` is blocked, no workloads inside the cluster will have access to abuse Tiller.

# Questions

If you have any questions, I can be reached via the following:

**Kubernetes Slack:** [`@v1k0d3n`](https://app.slack.com/team/U0A333J23)<br>
**Twitter:** [`@v1k0d3n`](https://twitter.com/v1k0d3n)<br>
**LinkedIn:** [`@bjozsa`](https://www.linkedin.com/in/bjozsa/)<br>

**Follow us:** [`@projectcalico`](https://twitter.com/projectcalico)

If you would like to have a workshop on [Helm](https://helm.sh/), [Calico](https://www.projectcalico.org/), [Istio](https://istio.io/), [Kubernetes Admission Controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/), or perhaps how each of these things can work together, please reach out and we can work on scheduling something for your area!

# Credits

The amazing [Helm team](https://kubernetes.slack.com/messages/C51E88VDG) on [Slack](kubernetes.slack.com) (especially [@bacongobbler](https://app.slack.com/team/U0D3ENG3G) and [@jdolitsky](https://app.slack.com/team/U4GDWJ5FB) for his contrinbutions to Helm's new push/pull mechenisms in v3).<br>
[@guslee](https://kubernetes.slack.com/messages/DNCFYTYK0) for his great article on Helm Security, for which this demonstration was inspired from.

# Final Thoughts

This repo will be maintained and updated as required. At the time that I make this repo public, I will continue to improving the v2 and v3 Chart examples (leveraging the `kubernetes-common` library), but everything else will work as it did during our demo.

I encourage you to explore [Helm v3](https://v3.helm.sh/docs/) and give feedback to the development team! Again, thanks for coming to the meetup, and hopefully we'll see you again soon!
