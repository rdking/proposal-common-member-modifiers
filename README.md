# proposal-common-member-modifiers
A proposal for adding `readonly`, `final`, and `override` as a standard means of modifying property descriptors for `class` members 

---
## Motivation

By one proposal or another, we will eventually have private data members in an ES `class` definition. Along with that, we will also likely have public data members. At the same time, it also looks like decorators will be released in the same time frame. This will make nearly anything that can be done by modifying a property descriptor possible. One of the things this will make possible is the ability to mark a data member as read-only. This will almost certainly become a fairly common operation. That means fairly early on, as developers become accustom to using decorators, several versions of a read-only type decorator will be released. They may have different formats and different names, but they will all be doing the same job, setting the `writable` attribute of the property descriptor to `false`. Instead of waiting for the confusion that will ensue, why not simply provide these things this upfront?

---
## Goals

The idea is to provide a definition prefix, much like static, that clearly defines the `writable` and `configurable` status of the field being defined. It is also a goal to make it clear when it is the intention of a subclass to completely supercede some functionality provided by a base class through accessors.

---
## Overview

Below is an example of the 3 modifiers given the now current classes-1.1 syntax:

```js
class A {
  //This stuff is private (in the closure)
  let static instances = 0;          //Private to the class
  let instanceNo;                    //Private to the instance
  let forShow = true;
	
  //This stuff is public (on the instance)
  final get originalInstanceNo() { return instanceNo; }
  get instanceNo() { return instanceNo; }
  get forShow() { return forShow; }
  set forShow(v) { forShow = v; }
  readonly prop1 = 42;               //Class default assignments in the definition
  final prop2 = "Change it's value all you want, but you'll never redefine this field.";
  final readonly prop3 = "Even my owner can't change me, value or otherwise!";

  constructor() {
    instanceNo = ++A::instances;     //Instance-specific assignments in the constructor.
  }
}

var a = new A();
a.instanceNo; //returns 1;
a.originalInstanceNo; //returns 1;
++a.prop1; // returns 43;
a.prop1; // returns 42;
a.prop2 = "I changed it!"; //returns "I changed it!"
a.prop2; //returns "I changed it!"


class B extends A {
  let instanceNo = Math.PI;

  override get instanceNo() {}
}
```