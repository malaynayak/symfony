# Building RAD

All right, we've done a lot of work. I've showed you the ugliest parts of the new system so that you can solve your real problems.  So, so far, it probably hasn't been that much fun. So I want to take an opportunity to create something new, so we can see how good this system feels. So I'm gonna create an event subscriber that's gonna add a new header to every single response. So in my AppBundle, I'll create an EventSubscriber directory, though that can be called anything. Inside there a new PHP class called AddNiceHader. Inside there a new PHP class called "AddNiceHeaderSubscriber".

Okay. Subscribers always work the same way. They implement EventSubscriberInterface, which means I can go to the code generate, or Command N menu, click implement methods, and have it add me the getSubscribedEvents. Here I'll return an array of KernelEvents, : : response. That's the name of the event I wanna listen to. And we'll have a : method called "onKernelResponse". Easy enough. And up top I'll create that public function onKernelResponse, and listeners to this event are passed a FilterResponseEvent object. It's all specific to the event system. Inside of here just add a header, I can say "event getResponse -> headers -> set". We'll set X-NICE-MESSAGE, we'll set to that was a great request. So there it is, 23 lines of code, we've touched only this one PHP file, and guess what, it works automatically. I'll go do inspect element on the page, go to network, refresh one more time, make sure I'm on all, and we'll click the main request, go to headers, and there is our X-NICE-MESSAGE. We just added an event subscriber to Symfony without doing almost anything. The service is automatically imported, automatically registered, and then it's automatically tagged thanks to auto configure.

All right so next let's suppose we want to log something inside of here. So we know this is a normal service so we're gonna use dependency injection. The question is what type-hint should we use for the logger? Well let's go over here to bin console debug container -- types and just to save us a little bit I'll correct that case insensitive to log. We find out there is ... And I'll search for logger ... Dammit. Bin console debug : container -- types, well whoa nothing is found. And that actually means right now there's no way for me to auto wire the logger, at least not without using deprecated functionality. So what's going on here? And actually the answer is that this auto wiring stuff is so new that some of the bundles are still catching up and adding aliases for their interfaces. Now Monolog has updated, has been updated, but it's available on version 3.1. So I'm actually gonna change this to ^ 3.1, and then I'm gonna run a composer update so that I can pull down the latest changes.

Now if you are having problems with this with other libraries and they haven't been updated yet, you can always add the alias yourself. In this case the type-hint should be for the PSR/Log/LoggerInterface, and you can set that as an alias to the at logger. And this is the great thing about auto wiring, you have control to do things you don't have to wait for the end user bundle to do it. In this case as soon as it finishes I'll run debug : container again and now we have PSR Log LoggerInterface. So I'll copy that, so I know that is gonna be my type-hint. So let's flip over, I'll do it, I'll do command N to make a constructor very quickly. Type-hint LoggerInterface logger, use alt enter to initialize my fields, just a shortcut for adding the property and setting it. And down here we can say this -> logger -> info, adding a nice header. And we've still only touched this file. So I'll go back, refresh. Click any of the links down here, go to the logs, and there it is. So now the logger thanks to auto wiring was automatically injected without any configuration.

So let's keep going. Let's say instead of hard coding this message we want to use our message manager. So, we will use MessageManager. I use alt enter once again to initialize that field, automatically set the property for me. So down here I can just say "message = this -> MessageManager getEncouragingMessage". And we'll use that below. And we know this is gonna work because our service id is equal to that type, exactly. So no surprise if we go back, refresh ... I'll actually click the logs icon this time. And you can go to Request/Response and click response, this is another way to see the headers. And there it is, X-NICE-MESSAGE "Woot". So it automatically put in our message generator, again, we're just working in on our business logic inside of here, all the configurations being automated for us.

So let's do something that's not gonna work. Let's add a third argument here called "showDiscouragingMessage". I'll use alt enter again to set that onto a new field. This is actually gonna be a Boolean, true or false, and notice it does not have a type-hint, although you can use a scalar type-hint if you want to, but that's not gonna help auto wiring. And down here, I'm gonna say "this -> showDiscouragingMessage" and if so we will use the messageManager getDiscouragingMessage and else we'll use the EncouragingMessage. So again we're just working inside of this file, so far Symfony's taken care of all the configuration for us. So now let's go back, refresh, and this time on any page we refresh we get the very clear error "Cannot autowire service addNiceHeaderSubscriber argument showDiscouragingMessage of method construct must have a type-hint or be given a value explicitly". So you don't configure anything until Symfony tells you that you need to configure something. At this point I can copy that class name, go into our services.yml and I know that we need to explicitly configure this service. So I'll put the service name, I'll add arguments and then we'll use the named argument format. Which means I'll copy showDiscouragingMessage here and say true. Now I'm gonna flip back. It works. And if we click down here and go to Request/Response, there is our not so nice message "We are never going to figure this out".

All right guys that is it. The way that you wire services behind the scenes, it's all still the same. It's all still about telling Symfony what services you have, what arguments that you have. It's just that before Symfony 3.3 you always had to be perfectly explicit. You needed to register every single service explicitly, and you needed to explicitly wire every argument and every tag. In Symfony 3.3 we try to automate that as much as possible. We try to pass in arguments whenever we can, or throw a clear exception. We also try to tag things for you automatically when we can. This also ... As a side benefit to this, this might feel like magic or less clarity in some ways but it's actually a step forward because it allows us to stop using our service container as a service locator. Meaning, we're no longer saying container -> get. This means that when Symfony builds its container cache it's aware of all the services that were being used in a container, and if there is any problem with any of them, even if it's a service your not using on that page, you're going to see a very clear compile time exception, and that is huge.

So I think, upgrading a project to this new format, you don't have to do it, and it does take a little bit of work. And it's not perfect, dealing with classes that have ... A class that has multiple services for it. Means that you don't get to take advantage of some of the nice features but overall I hope you love this, I think it's going to accelerate development while minimizing the WTF moments. All right guys, see you next time.


