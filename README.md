# The Singleton (Skelleton) Container

This document describes a programming standard for class initiation and dependencies.

The goal set by the `Singleton Standard` is to standardize and re-think how frameworks and libraries make use of
instancing and scope to resolve & obtain objects, parameters, and add dependencies. 

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119][].

The word `abstraction` in this document is to be interpreted as the extending TRAIT
to be used as `Psr\Singleton`. Let `Singleton`, `Skeleton`, `Psr\Singleton`, and `abstraction`
be synonymous for the entirety of this document.

The phase `Global Scope` in this document is to be interpreted as the container for 
variables and closures. In the example presented below, our `Global Scope` will be the 
global scope which stores our data; however, this SHOULD NOT be considered the only valid implementation.

[RFC 2119]: http://tools.ietf.org/html/rfc2119

1. Specification
-----------------

### 1.1 Basics

- The `Psr\Singleton` is a trait designed for speed and modularity 
  while encouraging semantically pleasing code. The main objectives focus to allow fluid 
  transfer of data from one object to another while eleminating unneeded class initiations. 
  The Skeleton design should be considerd a `Self Containment System` and could be 
  classified as an all purpose Container. Due to the modules major use of private and 
  static members, a stand-alone interface is not a valid stopping point for a Singleton Standard. 

- All modifications of the RECOMMENDED standard SHOULD NOT effect the abstraction's features 
  further described in this document. 
  
- Unwanted features MAY easily be overridden for any class that chooses to inherit `Psr\Skeleton`.
  You can override any method defined by the abstraction simply by redeclairing the methods name 
  in the child's class body. Note, the trait will override any extended class methods that 
  share a method name. The abstractions code be MUST follow current PSR return types and features.
  
- Singleton allows classes to be overridden by storing the new object in the 
  `Skeleton` static variable ( see examples ). Proceeding calls to the overridden 
  class will then be reflected.

- The RECOMMENDED Example uses the GLOBAL scope as a storage container to 
  eleminate further initiation time; however, other implementations MAY be used
  to change the storage location (see extensions). 
   
- You MAY OPTIONALLY use the abstraction's Skeleton Pattern to instantiate any class that 
  uses `Psr\Singleton`. The extension relies on the __call() and __callStatic() magic  
  methods, which help handle all requests. Methods MUST be declared as private to bind requests 
  to `Psr\Singleton`. Classes instancing is OPTIONAL and should only be used when needed. 
  
- The `Skeleton` reserves the following variable names which SHOULD NOT be used in the supporting class.

    `getInstance`
    `methods`
    `storage`
    
- When using `Psr\Skeleton` the following methods are made available: 

    `getInstance` 
    `__callStatic` 
    `__call`
    `Skeleton`
    `addMethod`   
    `__get`     
    `__set`    
    `__isset`    
    `__unset` 
     `set`   
     `get`   
     `has` 
    `__invoke` 

      
- `getInstance` takes all passed parameters and reflects it to the new instance's constructor.
   If the `getInstance` static variable isset, then it MUST be returned. Otherwise, this 
   Method will initiate the class and store the newly created object in the static `getInstance` 
   variable. This MUST be declaired as a public static function and return the `getInstance` object.

- `__callStatic` takes in a function name and its arguments: for coustom closures, privately 
   declaire methods, or public closures. Using the static operator for private methods will 
   attempt to run the `getInstance` method then pass its' arguments to `useSkeleton` method. 
   This may return anything (a *mixed* value). Note, if the method name is not found in an 
   scope the `useSkeleton` SHOULD throw an Exception. You should only call a class statically 
   once-per-method ( see usage ).

- `__call` takes in a function name and its arguments: for custom closures, privatly declare 
   methods, or public closures. Passed arguments will be sent to `Skeleton` which will be 
   returned (a *mixed* value).

- `Skeleton` takes two unique parameters: a method name and supporting arguments.  
   If a method is found the passed arguments will be reflected. If the requested method results 
   in null or void, then the current object will be returned. The abstraction MUST first 
   attempt to see if any custom methods (stored in the methods varible) have been defined 
   during the current execution. If not found, the `Skeleton` will then check if a method
   (type private) exists within the current scope. Finally, if still not avalible
   `Skeleton` will attempt to see if there exists a closure defined in the public scope 
   ( $closure[$name] ) with the given name (see `addMethod` for further details). This may return 
   anything (a *mixed* value) EXCEPT null. If null is returned from a public closure, the 
   called object will be returned. Note, if the method name is not found in an scope the 
   `useSkeleton` SHOULD throw an Exception. 

- `addMethod` takes two unique parameters: a function name and a valid closure.
   This function utilizes binding to allow closures to inherit the current scope.
   Adding a closure as a method allows the use of any variable within scope (aka $this).
   When called with `useSkeleton`, If the closure has a no return (or returns null) 
   the return is `$this`.
  
- `set` takes one unique and one OPTIONAL parameter: a varible name which will store 
   the value of the second parameter. If the second argument is null, the value of the
   (private) `storage` variable will be used. This functions is valid for creating new 
   instances (see usage). Return SHOULD be void.
   
- `get` takes one OPTIONAL parameter: the varible's name to be returned.
   If the parameter is null, the value of the (private) storage variable will be used. 
   This functions is valid for creating new instances (see usage).
   
- `has` takes one unique parameter: a variables name. This method will run issset() on 
   "$this->$variable". Note, if not defined in the current scope, `__isset` will be invoked
   to check if the variable exists in the $GLOBAL scope. 
   
- `__get` takes one unique parameter: a variables name. This magic method will return 
   the (mixed) value of the variable in the GLOBAL scope. If the variable does not 
   exist a new Exception SHOULD be thrown.

- `__set` magic method operates exactly the same as `set`. 
   By magic method design the second argument, however, will not be null.

- `__isset` magic method takes one unique parameter: the variable's name to check. 
   This functions uses `array_key_exists` to see if the requested variable exists in 
   the $GLOBAL scope. The return will be of type BOOL.
   
- `__unset` magic method takes one unique parameter: the variable's name to unset. 
   This will run 'unset' on '$GLOBALS[$variable]'. The return will be void.
   
- `__invoke` magic method takes no parameters. This method will return (a *mixed* value) the 
   private member $storage. 
   
- The continer also reserves the variable `$closures` (of type array) in the `$GLOBALS` scope.

### 1.2 Exceptions

Exceptions directly thrown by the container MUST implement the
[`Psr\Container\Exception\ContainerExceptionInterface`](#container-exception).

A call to the `get` method with a non-existing id SHOULD throw a
[`Psr\Container\Exception\NotFoundExceptionInterface`](#not-found-exception).

### 1.3 Core Concepts

The Singleton Container allows data to be efficiently stored and retreived without 
the need to pass large arrays through the stack or use of a Service Locator. Many
large classes use a constructor to initiate its data; however, this model means that
that the object may only be used in the scope it was defined in, or must be passed to 
other support objects.

Using a Skeleton system ensures that all data or procedure produced from ANY class 
can be looked up ( or called ) without the need of re-instance the object ( new $class ).

For example, Singleton patterns are especially useful for ensuring only one instance, 
or one connection of a database is active at a time. Standardizing this model will 
call for programmers to develop in a more pragmatic way. This differs from the 
common 'one-and-done' principle wich descibes using all, or needed functionallity 
immediately when an object is initiated (generally smallers more operation specific classes). 

The built in data container also allows for less redeclaration of variables as 
arguments-to-methods in the stack. This increases run time.

### 1.4 Skeleton Usage

Any class may use this pattern by adding the Singleton trait:

```php
<?php

namespace Example;

use Psr\Singleton;

class ExampleClass {
    use Singleton;
    
    public $scopeResolution;
    
    public function __construct()
    {
        $this->scopeResolution = 'World';
    }
    
}

Example\ExampleClass::newMethod('helloWorld', function ($print) {
    print "$print $this->scopeResolution";
};)->helloWorld('Hello');

```

This example shows off the symantics. 
"All we need to do is make sure we keep talking."

```php
<?php


// This would instanciate the Modules\Route class and set a default url (route) if nothing is matched
$route = Modules\Route::getInstance( 'easy/url/paths/' );

$closures['simiGlobals'] = function () {
    print ' we keep ';
};

// The '*' is a wild card
$route->match( '{variables}/{optional?}/*', function ($variables, $optional) {

    $this->variables = "Can easily route urls \n";

    $this->optional = "Optional variables can be defined by a '?' \n";

    $this->anywhere = "Easy data management; Not defined, no problem!";
    
} );


print PHP_EOL . $optional . PHP_EOL .
    "Know what works\n";

echo 'what doesnt\n' . PHP_EOL
    . "$variables \n$anywhere \n";


$pinkfloyd = 'is make sure'; //

// Don't like my Methods, Replace them
// then call then.. in the same line.. in the same scope ;) 

Modules\Request::addMethod( 'name', function ($arguments) {
    print "$arguments\n";})->name('Flexible Formatting Without Parsing' . PHP_EOL);

Modules\Request::name( "Closures from Object or Static Context\n" )->addMethod( 'party',
    function ($what) use ($pinkfloyd) {
        print "All we need $what $pinkfloyd";
    })->party('to do')->simiGlobals();

print "talking. \n";

```

Further/better implementation examples can be viewed at the (work in progress) CarbonPHP fraimwork.
( https://github.com/RichardTMiles/CarbonPHP )  


2. Package
----------

The abstraction as well as relevant examples can be found at (https://github.com/RichardTMiles).
[Psr/Singleton] (https://github.com/RichardTMiles) package. (still to-be-created)


3. Special Thanks
----------

To my older sister Morgan for getting detention to bring me my planner (and for always having my back). 
To my girlfriend Sam who listens to me talk about things she doesn't understand.
