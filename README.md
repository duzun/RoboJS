NOTE. this is more an experiment than a production code. 
It is inspired by Robotlegs. (the mvc bundle package) 
It started as a sample of how MutationObserver works.


#How it works.
You set a `data-mediator` attribute with an ID (whatever you want)

    <div data-mediator="mediator-a">a-2</div>
    <div data-mediator="mediator-b">b-1</div>
    <div data-mediator="mediator-c">c-1</div>

in `MediatorsMap.js` you define an Array that maps an ID and a Mediator path

``` 
[
    {
        "id": "mediator-a",
        "mediator": "client/MediatorA"
    },
    {
        "id": "mediator-b",
        "mediator": "client/MediatorB"
    },
    {
        "id": "mediator-c",
        "mediator": "client/MediatorC"
    }
]
``` 
    	
For instance in this sample I mapped 3 different Mediators.

When the builder finds a match between a `data-mediator` attribute and an ID from `MediatorsMap`, 
it will create a new instance of Mediator, storing the DOM Node into a property named `element` and executes `initialize` method

###Example:

```    
// Application.js

var MediatorsBuilder = require("../../src/org/display/MediatorsBuilder");
var MediatorsMap = require("./MediatorsMap");

// create an instance of MediatorsBuilder passing the map of Mediators.
var builder = new MediatorsBuilder(MediatorsMap);

/*
 * get the mediators and return a promise.
 * The promise argument is an Array of Mediator instances
 *
 */
builder.getMediators().then(function (mediators) {
    console.log("Mediators loaded", mediators);
}).catch(function (e) {
    console.log(e);
});

/*
 * when new DOM nodes are added to the document MutationObserver notify it, and a onAdded Signal is dispatched.
 * The Signal argument is an Array of Mediator instances
 *
 */
 
builder.onAdded.add(function (mediators) {
    console.log("Mediators added async", mediators);
});

/*
 * when new DOM nodes are removed from the document MutationObserver notify it, and a onRemoved Signal is dispatched.
 * The Signal argument is an instances of Mediator.
 *
 */
 
builder.onRemoved.add(function (mediator) {
    console.log("Mediators onRemoved async", mediator);
});
```
    
#EventDispatcher Class.
The `EventDispatcher` is your messaging System. It dispatches and listens to `Events` from your Application. 
It's meant to be a Singleton ( EventDispatcher.getInstance() ) in your application, but you can instantiate it as a normal Class.

#Mediator Class.
Mediator is the context where your logic runs for a specific bunch of DOM.
When a `data-mediator` matches an ID from MediatorsMap a new instance of Mediator is created. the DOM element is stored into a property named `element` and the `initialize` method is invoked.

Mediator has a reference to `eventDispatcher` too. 
This way it can dispatch / listen to messages in your application.
`addContextListener` and `removeContextListener` are responsible to map and unmap an event registered in EventDispatcher. `dispatch` is responsible to send an event with EventDispatcher.

Usually events are registered in `initialize` function

```
// 'event-name' is a String.
// this._handleEvent is the listener function.
// this is the scope of listener.
this.addContextListener("event-name", this._handleEvent, this);
```

To remove the listener you can do 

```
// 'event-name' is a String.
// this._handleEvent is the listener function.
this.removeContextListener("event-name", this._handleEvent);
```

When DOM element is removed from DOM Tree, Mediator removes all listeners registered to EventDispatcher and the `destroy` method is executed.  


###Concrete Mediator

You have to sub-class Mediator Class in order to code your logic. For example i define `MediatorB`. 

No matter how you implement inheritance. I just played with Vanilla-js to keep as cleaner as possible 


```
// MediatorB.js

var Mediator = require("../../src/org/display/Mediator");

function MediatorB() {
    Mediator.apply(this, arguments);
}

MediatorB.prototype = Object.create(Mediator.prototype, {
    constructor: {
        value: MediatorB
    },
    initialize: {
        value: function () {
            console.log("MediatorB", this.element);
            /**
             * a new listener is added.
             *
             */
            this.addContextListener("event-name", this._handleEvent, this);
        }
    },
    _handleEvent: {
        value: function (e) {
            console.log("_handleEvent", this);
            // after the first fired event, the listener is removed.
            this.removeContextListener("event-name", this._handleEvent);
        }
    },
    destroy: {
        value: function () {
            console.log("destroy");
        }
    }
});
```

	