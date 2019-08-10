---
layout: post
title: Watch for kubernetes resources changes and trigger handlers using kwatchman
---

What is the real [Cost of Change](https://www.newnettechnologies.com/whitepaper/nnt-whitepaper-problem-with-itil-change-management-process.pdf) ?

> *According to research from Gartner Group, “80 percent of unplanned downtime is caused by people and process issues (changes)”.*

Back in time as SRE on call duty, I was looking for a way to known when something have changed within my kubernetes clusters in a "passive" and completely automated way, changes such as number of replicas, container image versions, really anything that could be relevant and end up impacting production.

After a quick research I found [kubewatch](https://github.com/bitnami-labs/kubewatch) from bitnami to be potential solution, however kubewatch is not implementing the feature that I needed (manifest changes) and is in general quite noisy, after a couple of tries to contribute and get the feature I needed I realize that despite being a popular reposity with over 900 stars it was not maintained properly and had several problems, making it very difficult for me to implement such a large feature, the perfect excuse to build and implement my own solution from scratch and even improve it !

[kwatchman](https://github.com/snebel29/kwatchman) is a custom kubernetes controller that list and watches cluster resources using shared informers to then process logical changes in manifests through a chain of handlers.

Essentially you deploy kwatchman into your cluster of choice, you can restrict to a namespace and set of [label selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/).

Every time there is a change into kubernetes resources manifests an event is generated and passed through the configured chain of [handlers](https://github.com/snebel29/kwatchman#handlers) triggering powerful events.

For example to be notified on every change being deployed to your deployments and service on [slack](https://slack.com) simply [Install kwatchman](https://github.com/snebel29/kwatchman#installation) using its helm chart.

> :warning: To get notifications in slack you have to [configure it first](https://github.com/snebel29/kwatchman#the-slack-handler)

Please note that the default values does not include the slack handler, so you will have to configure it, uncomment the [following lines](https://github.com/snebel29/kwatchman/blob/e51de050e1662b8e1a03812b6d24d799ae00f573/build/chart/kwatchman/values.yaml#L31-L37) within the helm chart values.yaml file then redeploy with the new changes

```
$ helm install -n kwatchman .
```

Of course you can customize other values and even create your own handlers!

Changing the number of replicas in the cluster would cause kwatchman to trigger a notification in slack

<img src="https://raw.githubusercontent.com/snebel29/kwatchman/master/img/demo.gif">

The strategy with kwatchman is to focus on change management, providing semantic differences by creating a structure holding the list of changes that would available to the handlers for applying logic based on them, such as if this thing changed do this, etc.

That way teams can streamline deployment events of their projects for troubleshooting, and even machines could leverage them for root cause analysis by logging your relevant changes.
