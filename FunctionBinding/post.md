The <i>this</i> keyword is always important in JavaScript. I'll admit that it took me some time to actually understand how it works. You know, sometimes JavaScript is all about the scope. Where you are and what you have an access to. This article is about the <i>[bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)</i> function. Something which I use very often.

## The problem

We use closures all the time. While we define a closure and call it, it actually has its own scope and the variable <i>this</i> doesn't point to our original object. For example:

	var getUserComments = function(callback) {
	    // perform asynch operation like ajax call
	    var numOfCommets = 34;
	    callback({ comments: numOfCommets });
	};
	var User = {
	    fullName: 'John Black',
	    print: function() {
	        getUserComments(function(data) {
	            console.log(this.fullName + ' made ' + data.comments + ' comments');
	        });
	    }
	};
	User.print();

Let's say that <i>getUserComments</i> is an asynchronous functions. The result of the operation is return in a callback passed as argument. In the object <i>User</i> we are sending a closure and the result is:

	undefined made 34 comments

That's because the <i>this</i> in the result callback is not pointing to the <i>User</i> but to a newly created object. I've written a lot of code like this one:

	var self = this;
    getUserComments(function(data) {
        console.log(self.fullName + ' made ' + data.comments + ' comments');
    });

But it gets messy and fills the codebase with needless definitions of variables like <i>self</i> or <i>that</i>.

## Setting the scope of a function

There is a way to set a scope of a function. The <i>apply</i> or <i>call</i> method. 

	var getLength = function() {
	    console.log(this.fullName.length);
	}
	getLength.apply(User);

If we call <i>getLength()</i> directly we will get <i>Uncaught TypeError: Cannot read property 'length' of undefined</i>. Which is absolutely normal, because there is no <i>fullName</i> property defined. However with <i>apply</i> we are able to say what is the <i>this</i> actually. We are also able to send parameters.

The problem here is that the function is actually executed. In the first example we don't want this to happen. We need to pass a function, which is run after some time. Of course we may send the scope along with the function, but this means using two parameters all the time. And here is the place of the <i>[bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)</i> method. This is a function which sets the scope, but doesn't execute the function.

## The native <i>Function.prototype.bind</i>

<i>[bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)</i> is supported natively by most of the browsers nowadays. You may use it in the recent versions of Chrome, Firefox, Opera, Safari and 9th version and up of InternetExplorer.

	var User = {
	    fullName: 'John Black',
	    print: function() {
	        getUserComments(function(data) {
	            console.log(this.fullName + ' made ' + data.comments + ' comments');
	        }.bind(this));
	    }
	};
	User.print();

Notice the binding at the end of the closure's definition. Nice and simple we are setting a scope. If we need to send some arguments we are able to do it by putting more parameters. Like for example:

	var User = {
	    fullName: 'John Black',
	    print: function() {
	        getUserComments(function(msg, data) {
	            console.log(this.fullName + ' made ' + data.comments + ' comments' + msg);
	        }.bind(this, ' and 2 blog posts.'));
	    }
	};

The predefined arguments are placed in the beginning of the parameters' list.

## Polyfill 

If you need to use <i>bind</i> in an older browser I'll suggest to use [the polyfill published in MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind).

	if (!Function.prototype.bind) {
	  Function.prototype.bind = function (oThis) {
	    if (typeof this !== "function") {
	      // closest thing possible to the ECMAScript 5 internal IsCallable function
	      throw new TypeError("Function.prototype.bind - what is trying to be bound is not callable");
	    }

	    var aArgs = Array.prototype.slice.call(arguments, 1), 
	        fToBind = this, 
	        fNOP = function () {},
	        fBound = function () {
	          return fToBind.apply(
	          	this instanceof fNOP && oThis ? this : oThis, 
	          	aArgs.concat(Array.prototype.slice.call(arguments))
	          );
	        };

	    fNOP.prototype = this.prototype;
	    fBound.prototype = new fNOP();

	    return fBound;
	  };
	}

The first line checks if the function is available. If it is then the polyfill is not registered. <i>typeof this !== "function"</i> check is necessary because there are cases where you may run the <i>bind</i> method agains something different of a function. <i>aArgs</i> variable keeps the arguments which you want to be passed to the bound function. As you probably know every function in JavaScript has the magic <i>arguments</i> variable. It is something like an array, so we are able to extract the unknown number of passed parameters with the <i>slice</i> method. For example:

	var f = function() {
		console.log(Array.prototype.slice.call(arguments, 1));
	};
	f(10, 20, 30, 40); // outputs [20, 30, 40]

<i>fToBind</i> is the function which we are going to execute. <i>fBound</i> is the function which is returned and which we could pass around freely. And of course wherever we call it it will be run with the proper scope and parameters. The polyfill successfully keeps the prototype chain and performs a check if a scope is actually passed.