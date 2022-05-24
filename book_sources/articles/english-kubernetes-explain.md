Kubernetes explain

```text
Hi everyone, my name is Sai Vennam
and I'm a developer advocate with IBM
Today, I'm back with another video
where I'm going to be talking about all things Kubernetes.
Kubernetes as an orchestration tool
allowing you to run and manage your container-based workloads
today, I want to take a high-level look
at  a reference architecture of managed Kubernetes services
and dive a little bit deeper about how you would do a deployment of your micro-services.
Let's get started here.
So, we've got here, sketched out, two sides of the puzzle.
On the left side here, we've got the cloud side,
and what we've got here is a very important component 
that's going to be the Kubernetes master.
The Kubernetes master has a lot of important components in it,
but the most important piece \
						that we want to talk about today is going to be the API server.
The Kubernetes API server running on the master is integral \
					to running all of your workloads and exposes a set of capabilities,
allowing us to define exactly how we want to run our workloads.
On the right side here, on the customer-managed side,
we've got our worker nodes,
which are all also Kubernetes-based.
There's one major component that i want to point \
									our running on every single Kubernetes worker node,
and that's going to be the kubelet.
The kubelet, essentially, is responsible for scheduling and \
						making sure apps are healthy and running within our worker nodes.
You can imagine that the master and the kubelet are \
												going to be working together quite often.
Let's take a step back.
Why would someone want to start using Kubernetes?
Well, maybe they have some micro-services that make up a cloud-native application.
As we all know, micro-services are talking to each other over the network.
To really simplify this example,
let's say we've got a frontend and a backend,
and those are the two components that we want to scale out and \
														deploy to that cluster today.
So, Kubernetes uses YAML to define the resources that are sent to the API server,
which end up creating the actual application.
So, let's get started with that by sketching out a simple YAML for deploying a pod.
A pod is a really small logical unit which allows you to run \
													a simple container in worker node.
So, we'll start with that.
Let's say we've got a pod, and what we need is an image that's associated with it.
Let's say that it's a container, 
we've already pushed up to Docker Hub,
and we'll use my registry for this one.
And, let's say the name for the application is just "f" for frontend - version 1.
And one more thing that we want to add here,
let's just say we've got labels.
Labels are very important,
and we'll talk about why in a second here,
but they allow us to define exactly what the type of artifact we've got here is.
So, for the labels, we'll just say the application is "f" for "frontend".
Alright, so we've got that created,
and what we want to do is push it through our process to get into a worker node.
What we've got here is kubectl.
Using that, we're gonna be able to deploy the simple manifest that we've got \
												and have it in one of our worker nodes.
So, we'll push the manifest through kubectl,
it hits the API running on the Kubernetes master,
and that, in turn, is going to go and talk to one of the kubelets 
because we just want to deploy one of these pods and start it up.
So, taking a look,
let's say that it starts it up in our first worker node here with the label \
											that we've given it: "application is frontend".
And one thing to note here: it actually does get an IP address as well.
Let's say we get an internal IP address that ends in a ".1".
So, at this point, i could SSH into any of the worker nodes and \
												use that IP address to hit that application.
So, that's great for deploying a simple application;
let's take it a step further.
Kubernetes has an abstraction called deployments,
allowing us to do something and create something called a "desired state".
So, we can define the number of replicas we want for that pod,
and if something were to happen to the pod and it dies,
it would create a new one for us.
So, we've got the pod labeled as "application is frontend",
and we want to say that we want to create, maybe, three replicas of that.
So, going back to our manifest here.
One thing we need to do is tell Kubernetes that we don't want a pod,
we want the template for a pod.
So, we'll scratch that out,
and we'll create a template for a pod .
On top of that, we've got a few other things that we want.
So, the number of replicas - let's say we want three.
We've got a selector.
So, we want to tell this deployment to manage any application deployed with \
																that kind of name here.
We'll say match that selector here.
Again, this is not entirely valid YAML -
I just want give you an idea of the kind of artifacts that Kubernetes is looking for.
The last thing that we've got here is,
What kind of artifact is this?
And this is going to be a deployment.
Alright, so we've scratched out that pod and we've got a new manifest here.
What it's going to do: we're going to push it through kubectl,
it hits the API server.
Now it's not an ephemeral kind of object -
Kubernetes needs to manage the desired state -
so what is going to do is,
it's going to manage that deployment for as long as we have \
												that deployment and we don't delete it.
it's going to manage that here.
So, we'll say that it creates a deployment, 
and since we've got three replicas,
it's always going to ensure that we've got three running.
As soon as we've got the deployment created, we realize,
Hey, something's wrong, we've only got one, we need two more.
So, what it's going to do 
is it's going to schedule out deploying that application
wherever it has resources.
We've got a lot of resources still -
most of these worker nodes are empty,
so it decides to put one in each of the different nodes.
So, we've got the deployment created,
and let's just say we do the same thing for our backend here.
So, we'll create another application deployment: application is backend.
And for this one, let's just scale it out two times.
So, we'll go here: "application is backend".
And everyone's happy.
Now we need to start thinking about communication between these services.
We talked about how every pod has an IP address,
but we also mentioned that some of these pods might die.
Maybe you'll have to update them at some point.
When a pod goes away and comes back it actually has a different IP address.
So, if we want to access one of those pods from the backend or even for external uses,
we need an IP address that we can rely on.
And this is a problem that's been around for a while,
and service registry and service discovery capabilities \
													were created to solve exactly that.
That comes built-in with Kubernetes.
So, what we're gonna do now is create a service to actually \
														create a more stable IP address
so we can access our pods as a singular application,
rather than individual different services.
SO, to do that, we're gonna take a step back here,
and we're going to create a service definition around those three pods.
To do that, we're going to need some more manifest YAML.
So, we'll go back here and create a new section in our file.
This time we've got a kind: service.
And we're going to need a selector on that.
Again, that's going to match the label that we've got here.
And, the last thing that we need here is a type.
So, how do we want to actually expose this?
But we'll get to that in a second.
By default, that type is going to be cluster IP,
meaning our service can be accessed from inside the cluster.
So, deploying that through kubectl,
it hits our master, goes over here,
and creates that abstraction we talked about.
We can say that we created another one for the backend as well.
So, what we get now is a cluster IP.
Let's just say "CLIP" for short -
and that's going to be an internal IP.
Say, it ends in a 5.
And then another cluster IP for our other service here.
And, we'll say that ends in ".6".
So, now we have an IP that we can use to reliably do \
										communication between these services.
In addition, the KubeDNS service,
which is usually running by default,
will make it even easier for these services to access each other.
They can just use their names.
So, they could hit each other using the name "frontend".
"backend" - or "F" or "B" for short.
So, we've got that and we talked about how now the services can talk to each other,
by using these cluster IPs.
So, communication within the clusters is solved.
To do that, what we'll need to do is define the type of this service,
and what we want is a load balancer.
There are actually other ways to expose, like node ports as well,
but a load balancer, essentially that it's going to do,
where this is internal to actual Kubernetes worker nodes,
we can create an external IP now.
And this might be, let's say, a 169 address.
And now, what we can do is expose that directly to end-users 
so that they can access that frontend by directly using that service.
We've talked about 3 major components here today.
We've got pods.
Pods, which are then deployed and managed by deployments.
And then, facilitating access to those pods created by those deployments using services.
Those are the 3 major components working together with \
										the Kubernetes master and all the worker nodes
to allow you to really redefine your DevOps workflow for deploying your applications \
													into a managed Kubernetes service.
i know we talked about a lot today
but we want to get into more in-depth topics in our future light boarding videos.
for example, something like deployments.
So, feel free to drop a comment below,
leave us any feedback,
definitely subscribe, 
and stay tuned for more light boarding videos in the future
and thank you so much for joining me today.
```