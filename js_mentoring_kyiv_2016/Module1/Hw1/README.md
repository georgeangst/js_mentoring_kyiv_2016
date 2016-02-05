patterns overview
===
pattern singleton
---
```javascript
;(function(//some code) {
    var app = this;

    app.start = function __start__() {
        
        //some code
        
        // Overwrite the start function so that it cannot be called again.
        // If it is called again, log the attempt to console.
        app.start = function() {
            if (console && console.trace) {
                console.trace();
            }

            console.log('Warning: Attempting to run start function more than once!');
        };

        return app;
    };

    // Lights, Camera, Action!
    jQuery(app.start);

}).call(//some code);
```
main application object returned, pattern used to prevent any attempts to run app after it was initialised for the first time.
same technique used for implementation of main modules: app.Page, app.User, app.Alert, app.Confirm, app.Prompt, app.ServerErrorModal and app.CheckLastPaymentOrderStatus

constructor pattern
---
used as a common way to create specific types of objects
by prefixing a call to a constructor function with the keyword "new", we can tell js we would like the function to behave like a constructor and instantiate a new object with the members defined by that function.

```javascript
app.modules.Page = new app.Page();
app.modules.User = new app.User();
app.modules.Alert = new app.Alert();
app.modules.Confirm = new app.Confirm();
app.modules.Prompt = new app.Prompt();
app.modules.ServerErrorModal = new app.ServerErrorModal();
app.modules.CheckLastPaymentOrderStatus = new app.CheckLastPaymentOrderStatus();
```

module pattern
---
pattern used to implement early mentioned modules app.Page, app.User, app.Alert, app.Confirm, app.Prompt, app.ServerErrorModal and app.CheckLastPaymentOrderStatus
each have public methods to work with.
```javascript
;(function() {
   var app = this;

   var Page = function Page() {

   };

   $.extend(Page.prototype, {
       initialize: function initialize() {

       },

       //some code

       getPreviousData: function getPreviousData() {

       }
   });

   app.Page = Page;

}).call();
```

observer pattern (publish / subscribe implementation)
---

implemented as module, details are provided in comments
```javascript
;(function() {

    var app = this;

    app.observers = {};

    app.noop = function() {
      return '';
    };

    app.triggeredEvents = [];


    // triggers an event and alert all observers of the event
    // eventType - the type of the event to trigger
    // eventOption - an object with relevant named parameters
    // returns pubsub object
    
    app.trigger = function __trigger__(eventType, eventOptions, context) {
       // Use a timeout to allow for events to be asynchronus
       setTimeout(function() {
           var observers = app.observers[eventType];
           var i;
    
           if (observers) {
               i = observers.length;
    
               while (i--) {
                   if (typeof observers[i] === 'function') {
                       observers[i].call(context, eventOptions);
                   }
               }
           }
       }, 0);
    
       app.triggeredEvents = _.union(app.triggeredEvents, [eventType]);
    
       return app;
    };

    // subscribe to an event. callback will be called and passed any
    // eventOptions when the event is triggered.
    // eventType the type of event to subscribe to.
    // callback the function called when even triggered
    // returns pubsub object.

   app.subscribe = function __subscribe__(eventType, callback) {
       var observers = app.observers;

       if (!(eventType in observers)) {
           observers[eventType] = [];
       }

       observers[eventType].push(callback);

       return app;
   };

    // unsubscribe from an event
    // eventType name of event subscribed to handler function that is currently subscribed (optional) 
    // returns pubsub object
    

   app.unsubscribe = function __unsubscribe__(eventType, handler) {
       var observers = app.observers[eventType];
       var i;

       if (observers && observers.length) {
           i = observers.length;

           while (i--) {
               if (!handler || observers[i] === handler) {
                   observers.splice(i, 1);
               }
           }
       }

       return app;
   };


    // subscribe to an event then unsubscribe after the event is triggered once.
    // eventType the type of event to subscribe to.
    // callback to be called when the event is triggered.
    // return pubsub object.


   app.once = function __once__(eventType, callback) {
       var wrappedCallback = function(eventOptions) {
           callback(eventOptions);
           app.unsubscribe(eventType, wrappedCallback);
       };

       this.subscribe(eventType, wrappedCallback);

       return app;
   };

}).call();
```
decorator pattern
---
used to implement custom functionality for magnificPopup plugin
```javascript
$.SpcMagnificPopup = {
    open: function open() {
        $.magnificPopup.open(arguments[0]);

        if ($.magnificPopup.instance.isIOS && Response.band(DEFAULTS.GALAXY_TAB2_PORTRAIT)) { // Fix for background on ipad when keyboard opened
            $.magnificPopup.instance.content
                .on('focus', DEFAULTS.SELECTORS.FORM_FIELDS, this.onOpenKeyboard)
                .on('blur', DEFAULTS.SELECTORS.FORM_FIELDS, this.onCloseKeyboard);
        }
    },

    onOpenKeyboard: function onOpenKeyboard() {
        $.magnificPopup.instance.bgOverlay
            .addClass(DEFAULTS.CLASSES.BG_ABSOLUTE)
            .height(elems.$body.height());
    },

    onCloseKeyboard: function onCloseKeyboard() {
        $.magnificPopup.instance.bgOverlay
            .removeClass(DEFAULTS.CLASSES.BG_ABSOLUTE)
            .css('height', '');
    },

    close: function close() {
        $.magnificPopup.close(arguments[0]);
    },

    getInstance: function getInstance() {
        return $.magnificPopup.instance;
    }
};
```
facade pattern
---
widely used while jQuery methods were implemented
jQuery method .extend() provides hidden functionality to merge the contents of two or more objects together into the first object
```javascript
$.extend(ServerErrorModal.prototype, {
  //
});
```

or lodash methods
lodash method .merge() ecursively merges own and inherited enumerable properties of source objects into the destination object, skipping source properties that resolve to undefined. 
```javascript
_.merge(app.modules, app.createModule($module));
```
jQuery .proxy method that implements another pattern known as proxy - functions that takes a function and returns a new one that will always have a particular context.
```javascript
bindEvents: function bindEvents() {
    app.subscribe(app.EVENTS.LOG_IN, $.proxy(this.updateComponent, this));
    app.subscribe(app.EVENTS.LOG_OUT, $.proxy(this.updateComponent, this));
},
```