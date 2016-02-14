So, you have decided that your infrastructure needs automatic provisioning. But which framework should you use? If you are new to the world of provisioning, it might be hard to know what to look for in a good framework. Here are some considerations you should be aware of before you make your final decision.

There are many good provisioning frameworks out there. Among the most mature ones at the moment are [Chef](https://www.chef.io/chef), [Ansible](http://www.ansible.com), [Salt](http://docs.saltstack.com/en/latest/), and [Puppet](http://puppetlabs.com). And there might very well be others worth considering too. So, how do you differentiate between them?

## Two aspects

As we see it, there are two main aspects to consider when evaluating a provisioning framework. 

![Two aspescts of a framework](https://bekkopen.blob.core.windows.net/attachments/e78b6072-d8f1-467e-adc5-04393f533ad6)

First we have the part you interact with directly, the DSL. And secondly there is the tool, which is largely defined by the model of provisioning it supports. By model we mean how the setup works: is it pull based or push based, and does it require a master node.

Lets start by considering the latter.

## The provisioning models

When considering the tool, we are actually considering different models for provisioning.

There are three different models to choose among: *pull*, *push via master*, and *masterless push*. Each model has its own pros and cons, you should decide which model is right for you depending on your specific needs. Since each model only has a few implementations this will narrow down your options.

Note that you might also want to mix models. Some people, for instance, use a pull based framework to manage the infrastructure and a different push based framework for application deployment.

Lastly, your choice of a model will affect what software you will have to install where. Some of the models requires you to have a dedicated master node, and some requires you to have some agent pre-installed on the servers. Some requires only an SSH key on the node. Some requires Python or Ruby to run. This is all extra complexity, and you should think about whether you really need it.


### The pull model

![Diagram of pull model](https://bekkopen.blob.core.windows.net/attachments/6925688e-012e-43dc-b242-f58c30b2755c)

The pull model is supported by *Puppet*, *Ansible*, and *Chef Solo*.

If you need your provisioning to be highly scalable, you probably want the pull model. Here you upload the latest configuration changes to the master provisioning node, which then simply stores it. It is the responsibility of each of your other servers to pull the master regularly and apply any updates. 

This leads to the drawback that you don't control when provisioning is done. You also need an agent pre-installed on each of the nodes. And you really need to make sure your master stays up, or you risk having an update reach only some of your nodes.


### The push model

![Diagram of push model](https://bekkopen.blob.core.windows.net/attachments/0b776322-c18a-4349-b92e-5970c62ec53c)

Both *Salt* and *Chef* supports push via master.

A disadvantage of the pull model is that you lose some control over when the changes are applied to your servers. If this is a major problem for you, move to a push based model. Keep the master node, and let it push changes to all nodes. This way we get changes out to all servers immediately, and can control the order of things if we wish. This often makes sense if you want to do deployment as a part of the provisioning.


### The masterless push model

![Diagram of masterless push model](https://bekkopen.blob.core.windows.net/attachments/dd663986-18e8-45af-8df2-edf4a19715e1)

*Ansible* is an example of a framework supporting masterless push out of the box.

The advantage of having a master node is having a single source of truth, at the cost of an extra infrastructural component. If the cost of that component is high enough you might want to get rid of the master node altogether. This simplifies the model, and allows you to apply changes to your servers directly from your local machine.

Masterless push means that two people provisioning at the same time might cause trouble. So be careful if you are trying to scale this model beyond a single team. Think [two pizza rule](http://www.businessinsider.com/jeff-bezos-two-pizza-rule-for-productive-meetings-2013-10?IR=T).

This might be more suited for smaller infrastructures (less than 10 machines), as there is no longer a master to mediate changes. It can get really slow really fast.

Note that even with the masterless push model, you might want to keep a dedicated server to provide a stable environment for initiating the provisioning of your production servers.


## The DSL

After determining which model is right for you, you still have to find the best framework supporting this model. Looking at the DSL, there are some questions you should be asking.

- **Is it [idempotent](https://en.wikipedia.org/wiki/Idempotence)?**

 	When running the provisioning code repeatedly, you want the result to be the same on every run. This should be a property of every serious provisioning framework by now, but just in case the one you are considering isn't idempotent: stay away!

- **How readable is it?**

	When you set up automatic provisioning of your servers, what you are really doing is creating a specific documentation over all your servers and environments as code. The easier the DSL is to read, the better your overview is, and less time is wasted checking things.

    This is especially important in a large organization where a lot of different people need to read and understand the code, or in a setting where you frequently need to train new people.

    In the ideal case, the DSL should be so easily understandable that even non-technical people get something out of reading it.

- **Which abstractions does it provide?**

	Make sure the framework you choose have abstracted away things you don't want to deal with.

	To give an example, you probably want to set up some users on your servers, but you don't want to specify the steps for doing so. You want to describe usernames, groups, and perhaps which shell they prefer, but not the exact commands to invoke to make it so.

	Sometimes you'll probably have to resort to listing some exact command lines to execute, but a good DSL will have predefined tasks for most of the things you need to do.

	One of the most important abstractions is the ability to detect state changes in e.g. configuration files and deciding what to do based on that. You should not restart services "just in case". Instead you should be able to state something like: "if `X` has *changed*, *reload* service `Y`".

- **Does it support parametrization?**

	Parametrizing your code and thereby separating the data from the logic has several benefits. 

	For one thing, extracting things out into variables and data structures can help keeping your code [DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself). When the logic is no longer tied to a specific set of values, and you might be able to reuse it more easily. You know that code you were planning to write to set up a reverse proxy? Now you can reuse it when you need a different proxy.

	Having your data independent of the DSL of the framework will also prove very useful the day you choose to change frameworks, as the data structures will be easily reusable, while you have to throw away the framework specific logic.

	And finally, no matter how nice and readable the DSL is, it can't beat pure data structures. 


Some of these questions might be more or less important depending on your situation. For instance, readability is very important if many new people will need to involve themselves in provisioning, but less so if your team is small and has low turnover. Make sure you adress the questions important to you.


## Conclusions

Which provisioning framework should you choose? The answer is, as always, *it depends*. We have presented some of the major points on which you should judge a framework, but it's up to you to make a decision suited to your needs. Remember to consider which model is right for you before looking at the DSL, as the model constrains your options down the road in a way that cannot be rectified by any DSL.

This article has, quite possibly, left you even more unsure about which framework to choose. But at least now you know which questions to ask, to help turn your unknown unknowns into known unknowns.
