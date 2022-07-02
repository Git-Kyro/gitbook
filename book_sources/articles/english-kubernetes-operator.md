
kubernetes operator explained

```text
Hi everyone, my name is SaiVennam
and I'm with the IBM Cloud team.
Today we want to talk about operators,
and no, I'm not actually talking about operation teams,
but instead the operators framework which can be used on kubernetes or openshift.
CoreOS actually introduced the operator framework back in 20126.
CoreOS is now a part of RedHat and IBM.
Operator framework is quickly picking up traction as it's a \
								greate way of managing complex kubernetes applications.
Now, before i jump into this,
we want to actually introduce what the kubernetes control loop is \ 
									because it's a core part of the operators framework.
In this video, we're going to be talking about things like deployments and pods,
so if you're not familiar with those be sure to check out the kubernetes \
										 explained video that i've done on those topics.
But ler's get started with exactly what the control loop is in kubernetes.
Now, essentially the way it starts,
the control loop is the core part of kubernetes,
it observes the state of what's in your actual cluster.
So, that's the first step: observe.
Next, what it's going to do,
kubernetes is going to double check that the state in the actual cluster versus the state \
                                                                  that you want it to be.
So it's going to do a diff.
And finally, it wants to resolve that diff by acting on it.
So, the last phase of the control loop is act.
Now, the control loop is core to how kubernetes works \
and there's a controller that basically acts on that for every default resource.
kubernetes comes with number of default resources.
let's see an example of deploying an application without \
												operators using these default resources.
So, as an end user,
essentially the first thing you're going to want to do is write up some YAML,
the specification for that actual application.
And, for our particular example, let's say that we're doing a deployment.
And in this deployment, we'll have to define some configuration.
Things like, what's the image? and maybe the replicas, and maybe some other configuration.
So, that's one kunernetes resourece.
And, essentially what you would do, is take that and deploy it into \
         					your kubernetes cluster at which point a deployment is made.
Here's where the control loop kicks in: so kubernetes observes the state of your cluster,
so we've got a kubernetes cluster here,
and checks for the difference between what you want versus what's there.
First thing it notices: there's no pods.
So, it's gonna act on that difference,
and it's gonna create some pods.
Now, let's say for a fairly complex application we don't just have \
													one YAMl but we have a secound YAML,
it's for hte backend,
and so that deploys in the second deployment,
and that in turn deploys a pod using the controllers in the control loop.
Now, it's simple example, but say you want to go through here,
scale up the application, make some changes,
set up some secrets, environment variables,
every single time you have to either create new kubernetes resources,
or go in here and edit hte existing ones.
that can start to get fairly difficult.
Now, ler's see how that's done in a world where we're using operators.
Now, the first thing you would need to do is install the operator itself.
So, someont on your team has to create the operator,
or maybe you can use one of the many that are our there on operatorHub,
or the community is building on the operators that are available.
So, the first thing you need in your kunernetes cluster is tht OLM,
which is the operator lifecycle manager,
which basically manages operators that you have installed.
Next, you deploy your actual operator into the cluster.
the operator is made up of two major components.
The first componment in an operator is going to be the CRD.
The other one is going to the controller.
Now, the CRD is basically a "Custom Resouece Definition".
So, we've talked about default resources - things like deployments and pods.
A custom resource is something that you define as a user is kubernetes,
or maybe an operator defines, it so that you can create YAML to work \
														against that costom config.
The controller is basiclly a custom control loop which run as a pod in your cluster
and runs this control loop against your custom resource definition.
let's say that operator is created for our custom application deployment here,
so instead of having to write multiple deployments and setting up config maps and \
												secrets for whatever our cluster needs,
we instead, as an end user, we'll just deploy one YAML.
Maybe we called this operator MyApp.
It could be a little bit more meaningful - we can call it stateful app, front-end app,
whatever we want it to be.
And then, we could define some config here,
or we can use default that are set.
we of have a choice of options here.
Then we take this operator and we deploy it directly into the cluster.
At this point the operator takes over,
and this is actually responsible for running that control loop,
and figuring out exactly what needs to be running.
So, it's going to realize that we need a couple of deployments and the pods.
Now, this is a kind of a format, or an approach to managing applications \
															that's inherently easier -
and scales better - than this approach,
because, as an end-user, you ready only have to worry about the config that's \
																	been exposed to you,
and the operator itself manages the control loop and the state of the application
- how it needs to look.
Now, there are great operators our there already,
things like managing etcd, or various databases,
or even IBM Cloud Services.
So, all of those operators currently exist on OperatorHub.
But say you want to develop your own, maybe a custom operator for something that \
												is native to your application architeture,
kind of like what we sketched out here.
Well, there's a number of ways you can do that.
And there's something called Operator SDK that allows you to start \
														building out operators yourself.
Now, I'd say the easiest way to get started with an operator,
is to use the Helm operator.
So, helm. as you may already know,
there is a video that where David Okun goes over wxactly how helm works.
Be sure to check that one out.
The helm approach allows you to take a Helm chart and apply that \
													 towards an operator, expose config-
so, it allows you to get to a fairly mature lavel of an operator.
Kind of something like this for a chart that's already there.
Now the maturity of operators, what I've sketched out down here,
falls into 5 different levels.
Now, Helm actually hits the first 2 levels of maturity.
Let's talk about what those levels are.
The first one is a basic install.
Essentially, the first level is basically going to allow you to do just \
												provisioning of the resources required.
Now, the second phase goes a little bit further it's gonna allow you to do upgrades.
So. this supports minior and path version upgrades to whatever is defined in your operator.
Now, Helm gets you that far,
what about for the next 3 levels of maturity?
For these, you're going want to use either GO,
or, you can also use Ansible.
Now, these will allow you to actually get to all 5 levels of maturity with operators.
Let's quichly talk about what those are.
At the third level. we've got full lifecycle support.
So, this is storage lifecycle, app lifecycle.
It's also going to allow you to do things like bakcup and failure recovery.
So, that's something that would be configured and developed into the operator,
who ever developed that one.
Fourth, what we've got here is insights.
This is going to allow you to get deep metrics, and analysis,
logging, that kind of thing, from your actual operator.
And finally, what we have is something called "autopilot".
And just as the name implies,
this is going to have a lot more functionality built into operator itself.
Basically it's going to allow you to do automatic scaling,
- horizontal and vertically.
It's going to do acutomatic config tuning.
If your operator-based app gets into a bad state,
it's gonna identify that automatically.
So, there are the 5 levels of maturity that operators can have.
By looking on OperatorHub, you can see the ones that the community has developed and \
												see what level of maturity that they hit,
and then, again, by using operator SDK,
you can build you own operators using either Helm, Go or Ansible.
Thanks for joining me for this video on operators.
If you have any questions be sure to drop us a line below.
If you want to see more videos like this in the future,
please like and subcribe.
And don't forget, you can always get started on the cloud at \
										no cost by siging up for a freeIBM Cloud account.
```