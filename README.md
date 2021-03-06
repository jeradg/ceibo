# Ceibo

JavaScript micro library to model trees that evaluate arbitrary code when
accessing its nodes.

The tree is modeled as a plain JavaScript object where each node has an
arbitrary getter function. This allows to have a representation of a tree where
a subtree is generated on the fly when a node is accessed.

## Examples

Let's start by doing the most simple case, the identity case:

```js
var root = Ceibo.create({
  foo: {
    bar: 'baz';
  }
});

console.log(root.foo.bar); // "baz"
```

You can create special node types called _descriptors_ that allow you to respond
to node access:

```js
var root = Ceibo.create({
  foo: {
    isDescriptor: true,

    get() {
      return 'bar';
    }
  }
});

console.log(root.foo); // "bar"
```

As you can see, a _descriptor_ is a JavaScript object that has a `isDescriptor`
attribute.

You can define a `get` method or you can declare a `value` attribute, then the
value attribute is going to be used as is:

```js
var root = Ceibo.create({
  foo: {
    isDescriptor: true,

    value(answer) {
      return `The answer to life, the universe and everything is ${answer}`;
    }
  }
});

console.log(root.foo('42')); // "The answer to life, the universe and everything is 42"
```

_descriptors_ can inspect and mutate the target object by defining a `setup`
method:

```js
var tree = Ceibo.create({
  foo: {
    isDescriptor: true,

    get() {
      return 'bar';
    },

    setup(target, keyName) {
      Ceibo.defineProperty(target, keyName.toUpperCase(), 'generated property');
    }
  }
});

console.log(tree.foo); // "bar"
console.log(tree.FOO); // "generated property"
```

Note that Ceibo trees are read-only, so you cannot reassign attributes:

```js
var root = Ceibo.create({
  foo: 'bar';
});

root.foo = 'baz'; // => throws an error!
```

You can redefine how each value type is processed when the Ceibo tree is
created:

```js
function buildString(treeBuilder, target, keyName, value) {
  Ceibo.defineProperty(target, keyName, `Cuack ${value}`);
}

var root = Ceibo.create(
  {
    foo: 'first value'
  },
  {
    builder: {
      string: buildString
    }
  }
);

console.log(root.foo); // "Cuack first value"
```

Redefine how plain objects are processed to generate custom attributes:

```js
function buildObject(treeBuilder, target, keyName, value) {
  var childNode = {
    generatedProperty: 'generated property'
  };

  // define current keyName and assign the new object
  Ceibo.defineProperty(target, keyName, childNode);

  // continue to build the tree recursively
  treeBuilder.processNode(value, childNode, target);
}

var root = Ceibo.create(
  {
    foo: {
      bar: 'baz'
    }
  },
  {
    builder: {
      object: buildObject
    }
  }
);

console.log(tree.generatedProperty); // "generated property"
console.log(tree.foo.generatedProperty); // "generated property"
console.log(tree.foo.bar); // "baz"
```

You can navigate to parent nodes

```js
var tree = Ceibo.create({
  foo: {
    bar: {
      baz: 'a value'
    }
  }
});

console.log(Ceibo.parent(tree.foo.bar).bar.baz); // "a value"
```

You can assign custom parents to trees

```js
var parentTree = Ceibo.create({ foo: 'value' });
var childTree = Ceibo.create({ bar: 'another value' }, { parent: parentTree });

console.log(Ceibo.parent(childTree).foo); // "value"
```

## License

Ceibo is licensed under the MIT license.

See [LICENSE](./LICENSE) for the full license text.
