# proposal-common-member-modifiers
A proposal for adding `readonly`, `final`, `enumerable` and `hidden` as a standard means of modifying property descriptors for `class` members 

---
## Motivation

By one proposal or another, we will eventually have private data members in an ES `class` definition. Along with that, we will also likely have public data members. At the same time, it also looks like decorators will be released in the same time frame. This will make nearly anything that can be done by modifying a property descriptor possible. 

Some of the most common things possible with property descriptors is changing the attributes of a property. By default, data properties will be declared enumerable, writable, and configurable, while function and accessor properties are denied enumerability. Instead of waiting for the community to define and agree upon decorators for these purposes, why not define them now?

---
## Goals

The idea is to provide a definition prefix, much like static, get, and set, that clearly defines the `writable`, `configurable`, and (for data properties) `enumerable` status of the data property being defined. In general, it should be completely unnecessary to use `Object.defineProperty` to modify the properties of a class member.

---
## Definitions

* `readonly` - sets `writable: false`. Not viable for accessor properties.
* `final` - sets `configurable: false`. Prevents<sup>1</sup> subclasses override of this member.
* `hidden` - sets `enumerable: false`. Used to make data properties non-enumerable.
* `enumerable`<sup>2</sup> - sets enumerable: true`. Used to make functions or accessor properties enumerable.

<div style="font-size: 85%;"><i>
1. The field can still be overridden during or after construction by temporarily removing the prototype.<br/>
2. Should only be applied to the getter of an accessor property.
</i></div>

---
## Overview

Below is an example of the 4 modifiers given the class-members syntax:

```js
class A {
  //This stuff is private (in the closure)
  let static instances = 0;          //Private to the class
  let instanceNo;                    //Private to the instance
  let forShow = true;
	
  //This stuff is public (on the instance)
  final get originalInstanceNo() { return instanceNo; }
  //These 2 accessor properties appear in Object.keys(this)
  enumerable get instanceNo() { return instanceNo; }
  enumerable get forShow() { return forShow; }
  set forShow(v) { forShow = v; }
  readonly prop1 = 42;               //Class default assignments in the definition
  final prop2 = "Change it's value all you want, but you'll never redefine this field.";
  final readonly hidden prop3 = "Even my owner can't change me, value or otherwise, and I'm not enumerable either!";

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

```

## Notes
* I'd much rather that `enumerable` not need to exist. However, since TC39 decided that `class` should hide accessor properties on the prototype, the situation begs for a declarative means of making accessors enumerable again. Something like this might be better left to a `class` decorator instead. 
* The class-members proposal is not a prerequisite for this proposal, only the preferred private member notation of this author. This proposal works equally well with class-fields.