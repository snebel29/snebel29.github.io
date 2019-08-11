---
layout: post
title: Watch for kubernetes resources changes and trigger handlers using kwatchman
---

What is the real [Cost of Change](https://www.newnettechnologies.com/whitepaper/nnt-whitepaper-problem-with-itil-change-management-process.pdf) ?

> *According to research from Gartner Group, “80 percent of unplanned downtime is caused by people and process issues (changes)”.*

Back in time as an SRE in on call duty, I was looking for a way to know when something has changed within my kubernetes clusters in a "passive" and completely automated way, changes such as number of replicas, container image versions, really anything that could be relevant and end up impacting production.

After a quick research I found [kubewatch](https://github.com/bitnami-labs/kubewatch) from bitnami as a potential solution, however kubewatch was not implementing the feature that I needed (report manifest changes) and is in general quite noisy, after a couple of tries to contribute and release the feature I needed I realized that despite being a popular repository with over 900 stars it was not maintained properly and had several problems, making it very difficult for me to implement such a large feature, the perfect excuse to build and implement my own solution from scratch and even improve it!

[kwatchman](https://github.com/snebel29/kwatchman) was born, it is a custom kubernetes controller that list and watches cluster resources using shared informers to then process those logical changes in manifests through a chain of handlers.

Essentially you deploy kwatchman into your cluster of choice, you can restrict watching to a given namespace or to a set of [label selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/).

Every time there is a change into kubernetes resources manifests an event is generated and passed through the configured chain of [handlers](https://github.com/snebel29/kwatchman#handlers) triggering powerful actions.

For example to be notified on every change being done to your deployments and services on [slack](https://slack.com) simply [Install kwatchman](https://github.com/snebel29/kwatchman#installation) using its helm chart.

> &#9888; To get notifications in slack you have to [configure it first](https://github.com/snebel29/kwatchman#the-slack-handler)

The default values file does not include the slack handler, so you will have to configure it, copy `values.yaml` locally from the [chart repository](https://github.com/snebel29/snl-charts/tree/master/kwatchman) and edit it.

Uncomment the [following lines](https://github.com/snebel29/snl-charts/blob/91b38d3e732ef0aee02c304b99499b254b4520ca/kwatchman/values.yaml#L31-L37) and add your webhook url, you can configure the resources that you want to watch as well [here](https://github.com/snebel29/snl-charts/blob/91b38d3e732ef0aee02c304b99499b254b4520ca/kwatchman/values.yaml#L19-L23), when you are ready redeploy kwatchman with the new changes.

```
$ helm upgrade kwatchman \
               --namespace kwatchman \
               --values values.yaml \
               snl-charts/kwatchman
```

Of course you can customize other values and even create your own handlers!

Still lost? this is how changing the number of replicas in the cluster would cause kwatchman to trigger a notification in slack

<img src="https://raw.githubusercontent.com/snebel29/kwatchman/master/img/demo.gif">

The strategy ahead with kwatchman is to focus on change management, providing semantic differences by creating a structure holding the list of changes that would available to the handlers for applying logic based on them, such as if this thing changed do this, etc.

That way teams can streamline deployment events of their projects for troubleshooting, and even machines (AIOPS) could leverage them for root cause analysis by logging your relevant changes.
