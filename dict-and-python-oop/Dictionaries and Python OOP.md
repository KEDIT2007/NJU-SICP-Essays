## Introduction

This article offers a look under the hood of OOP in Python. I will talk *a lot* about language-specific features that only a language as idiosyncratic as Python has. I will try to untangle the general idea of OOP from the implementation details of the Python language. However, the nature of the Python language makes the path unnecessarily winding, and we *will* be looking at implementation details that many tutorials skip. Keep in mind that many of these ideas may not apply directly to other languages.

Before reading this article, I assume that you're familiar with the concepts that have been covered in the first half of the course. For a quick recap, make sure you're comfortable with the following concepts:

- Python names and functions
- Frames and closures
- Basic types like `int`, `list`, `tuple`, etc. and related operations
- Mutable and immutable objects
- Dictionaries (we'll be using them a lot!)

Also, we will be taking examples from [our textbook](composingprograms.com), so I recommend reading our textbook first, but it is not strictly necessary.

If you're already introduced to the basics of Python OOP, I suggest that you throw everything about Python OOP away when reading this article, as this article is aimed at building intuition about Python OOP for someone who is completely new to this topic. If you have some knowledge about this topic, the knowledge could sometimes be a curse. 

It's important to note that this article is not a handbook of concepts and syntax, but an attempt to take a journey of struggles and dilemmas before we come to what a typical tutorial arrives at. It is you that are expected to summarize and internalize what you've learned in this journey.

## More Vending Machines

*This section contains examples from [2.4 Mutable Data in our textbook](composingprograms.com/pages/24-mutable-data.html) and [lab05](sicp.pascal-lab.net/2025/labs/lab05).*

In previous exercises, we've familiarized ourselves with various vending machines. Here is one example:

```python
def make_vending_machine(product, price):
    balance, stock = 0, 0
    def restock(amount):
        nonlocal balance, stock
        ...
        
    def deposit(amount):
        nonlocal balance, stock
        ...
    
    def vend():
        nonlocal balance, stock
        ...
    
    return restock, deposit, vend
```

	In this example, we maintain mutable state using closures and the `nonlocal` keyword. The **interface**, i.e. how we can perform operations on this vending machine, is the three functions returned by `make_vending_machine`. In other words, this vending machine is **represented by** the three functions.
	
	The drawback is immediately obvious: we would have to keep track of these three functions at the same time. This would easily become chaotic when we want more than one vending machine in our program. Imagine doing this:

```python
coke_restock, coke_deposit, coke_vend = make_vending_machine("Coke", 2)
pepsi_restock, pepsi_deposit, pepsi_vend = make_vending_machine("Pepsi", 2)
# and more!
```

	This will soon grow out of control.
	
	One immediate fix would be grouping these functions per vending machine. Or, simply don't unpack the return values:

```python
coke_vm = make_vending_machine("Coke", 2)
coke_vm[0](5) # restock 5 cokes!
```

	However, this is not good enough because `coke_vm[0]` is pure nonsense: what on earth does it mean to "take the first element" or "use subscript 0 on" a vending machine? The code itself lacks meaning. Or, it is not **self-explanatory**.
	
	The attempt above falls short because we are using integer subscription, and the integers themselves do not mean anything. We want to find a readable and self-explanatory way to organize these functions.
	
	Well, if subscripting by integers doesn't work, you might think, why not use more readable keys, like strings? `coke_vm['restock']` definitely looks better than `coke_vm[0]`. This is where dictionaries come into play.

```python
coke_vm = dict()
coke_vm['restock'], coke_vm['deposit'], coke_vm['vend'] = make_vending_machine("Coke", 2)

coke_vm['restock'](5)
coke_vm['deposit'](10)
```

	This immediately looks better! However, the second line still looks a bit messy, so let's just have `make_vending_machine` return a dictionary:

```python
def make_vending_machine(product, price):
    balance, stock = 0, 0
    def restock(amount):
        nonlocal balance, stock
        ...
        
    def deposit(amount):
        nonlocal balance, stock
        ...
    
    def vend():
        nonlocal balance, stock
        ...
        
    dispatch = {'restock': restock,
                'deposit': deposit,
                'vend': vend}
    
    return dispatch
```

	You may notice that this version closely resembles the code in our textbook. The returned dictionary serves as a **complete representation** of the vending machine. We call dictionaries like this **dispatch dictionaries**.
	
	If you compare the code snippet above with that provided in the textbook, you may find that the textbook example takes it one step further. Instead of using closure variables to maintain mutable state, it directly stores the mutable value(s) inside the dispatch dictionary. We may rewrite our code accordingly:

```python
def make_vending_machine(product, price):
	def restock(amount):
        # use dispatch['balance'] and dispatch['stock'] instead
        ...
        
    def deposit(amount):
        # use dispatch['balance'] and dispatch['stock'] instead
        ...
    
    def vend():
        # use dispatch['balance'] and dispatch['stock'] instead
        ...
        
    dispatch = {'restock': restock,
                'deposit': deposit,
                'vend': vend,
                'balance': 0,
                'stock': 0}
    
    return dispatch
```

	By doing so, we handle mutable state completely using dispatch dictionaries. However, this also **exposes** `balance` and `stock` to the user. That is, the user may simply write `coke_vm['balance'] = 100` to bypass the interface we intend. This is a noteworthy trade-off that alludes to an important concept in programming called **encapsulation**. However, we'd be too sidetracked to cover that in detail in this article.

## Dictionaries and Everything

*This section contains examples from [lab04](sicp.pascal-lab.net/2025/labs/lab04).*

Now that we have dispatch dictionaries at our disposal, we're tempted to refactor more code with this powerful tool.

	Before we write any code, let's first address the elephant in the room we just left behind: **exposed state**. Generally speaking, this is a sin. However, in this article, we simply don't worry about that, because we're consenting adults writing Python, and in Python, this is just business as usual.
	
	Now we are onto our ambitious attempt to refactor everything with dispatch dictionaries. Back when we were first introduced to ADTs, trees became a nightmare for many of us. Now it's time for a role switch! Let's put ourselves in the TAs' shoes and fashion trees from dispatch dictionaries (which might exactly be how the TAs knew whether you'd broken the abstraction!).
	
	Recall that a tree has a `label` and `branches`, which is a list of trees. Test your understanding of dispatch dictionaries by creating a dispatch dictionary that represents a tree. Here's one possible implementation:

```python
def create_tree(label, branches=None):
    if branches is None:
        branches = []
    dispatch = {'label': label,
                'branches': branches}
    return dispatch
```

	Next we may follow the example of vending machines and fit every operation on trees into this function, which may result in something like this:

```python
def create_tree(label, branches=None):
    if branches is None:
        branches = []
    def is_leaf():
        return not dispatch['branches']
    def print_me():
        ...
    def is_binary():
        ...
    dispatch =  {'label': label,
                 'branches': branches,
                 'is_leaf': is_leaf,
                 'print_me': print_me,
                 'is_binary': is_binary}
    return dispatch
```

	And the list of enclosed functions goes on. This looks fine at first glance: the functions work and the interface is readable. However, trees scale easily. Chances are that we get thousands of tree nodes in one program. With every node being created, all these enclosed functions have a new copy. Consequently, these copies of functions (or technically function objects)  would quietly eat up the memory.
	
	To fix this, we may resort to the most primitive approach we had established before we were introduced to closures. Since the only difference between the enclosed functions is the dispatch dictionary to be operated on, we can separate the logic and place the functions outside `create_tree`.

```python
def is_leaf(): # Does this actually work?
    return not dispatch['branches']

def print_me():
    ...

def is_binary():
    ...

def create_tree(label, branches=None):
    if branches is None:
        branches = []
    dispatch =  {'label': label,
                 'branches': branches}
    return dispatch
```

	Yet this is not enough. When these functions sit inside `create_tree`, they have access to the target dictionary (since they live in the same frame). When lifted out of that frame, they have no idea what `dispatch` is. Therefore, we would have to explicitly pass the dictionary as an argument.

```python
def is_leaf(tree_dict):
    return not tree_dict['branches']

def print_me(tree_dict):
    ...

def is_binary(tree_dict):
    ...

def create_tree(label, branches=None):
    if branches is None:
        branches = []
    dispatch =  {'label': label,
                 'branches': branches}
    return dispatch
```

	Technically speaking, we've transformed the dictionary to a pure **state dictionary**. The dictionary does not do any dispatching. Now if we want to check whether a tree node `tr` is a leaf, we write `is_leaf(tr)` , which looks okay, but it is inconsistent with how we would access the label of a tree (`tr['label']`). Also, it no longer feels like `is_leaf` is something that `tr` checks and returns, but rather there's some supervisor overseeing and controlling everything. If only we could still write `tr['is_leaf']()`!
	
	Well, it turns out that this struggle is so common that developers well before us have arrived at different solutions. Before we get to look at any of them, let's make a quick round-up.
	
	What we've touched upon in this section is actually the primitive form of Object-Oriented Programming (OOP). We treat everything as an object that has its own **attributes** (such as `label` of a tree) and **actions** (such as `deposit` of a vending machine). By now, we have been implementing this idea using dispatch dictionaries (or more specifically, a combination of state dictionaries and functions), and it seems that we're hitting a wall.
	
	We're now faced with a dilemma: if we put everything (attributes and actions) in a dispatch dictionary, function copies take up too much memory, and if we lift the functions outside the dictionaries, we detach action from the object (represented by a dictionary) and litter the global **namespace**.

> [!Tip]
>
> Namespaces are like dictionaries that map names to objects. By default, everything we write lives in the **global namespace**.

## Clean up Things with Classes

In the last section, I mentioned early developers have arrived at different solutions to our dilemma. In this section, we'll scratch the surface of the solution that Python has come up with: using **Classes**.

	In this section, we'll see how Python's designers have tried to solve the problem. Since we're desperate to find a solution, we'll be pragmatic for now and first look at what Python has provided for us with classes. After we're done with a more satisfactory version using classes, we'll look deeper beneath the surface.
	
	In essence, classes in Python are blueprints for dictionaries resembling what we've just crafted, but with a twist: instead of using brackets and string keys, Python introduces the dot operator (`.`). Where we write something like `vm['deposit']`, we write `vm.deposit` instead. Also, it allows us to organize our previously loose functions inside its namespace, preventing them from cluttering the global namespace. Cleaner syntax and no more floating functions, two problems solved in one shot!

> [!Tip]
>
> The dot operator is not a simple replacement of the dictionary subscription syntax, and they are mostly not interchangeable. When we write something like `vm.deposit` later, `vm['deposit']` won't likely work. However, it is a convenient mnemonic to think of the dot operator as dictionary subscription.
>
> Also, you can think of the dot operator as an "inverted of". For example, `vm.deposit` means "the `deposit` function of `vm`". In Chinese, a more direct analogy is "的".

	Let's get ourselves familiar with the `class` syntax through an example. Here is a piece of code taken from the last section with some modifications:

```python
def create_tree(label, branches=None):
    if branches is None:
        branches = []
    dispatch = dict()
    dispatch['label'] = label
    dispatch['branches'] = branches
    return dispatch

def is_leaf(tree_dict):
    return not tree_dict['branches']

def print_me(tree_dict):
    ...

def is_binary(tree_dict):
    ...
```

	Here, we want to reorganize our code using classes. To do so, we use the `class` keyword:

```python
class <Class_Name>:
	<Class_Suite>
```

	Think of the class block as a container. Remember those "floating functions" (`is_leaf`, `print_me`) that were cluttering up our global namespace? We can simply move them inside this block to keep things organized. Remember to replace subscription with the dot operation!

```python
class Tree:
	def is_leaf(tree_dict):
        return not tree_dict.branches
    
    def print_me(tree_dict):
        ...
    
    def is_binary(tree_dict):
        ...
```

	By doing so, Python organizes these function under **the class's namespace**, i.e. these functions belong to the `Tree` class. To access them, we also use the dot operation: `Tree.is_leaf(tr)`.

> [!Tip]
>
> At first glance, the dot operator is used differently than "accessing elements in a dictionary" I promised before. For now, let's think of it as two operations (retrieving objects in a certain namespace and accessing elements in a dictionary-like object) sharing one symbol. We'll later discover that these two operations are essentially the same.

	 "Wait a second!" you might say, "where is the `create_tree` function? And you promised we can write `tr.is_leaf()`!" 
	
	Let's first address the `create_tree` problem. If we now look closer at the `create_tree` function, we may notice that this function is actually doing two things: (a) creating a dictionary (an object) and (b) putting things into the dictionary using subscription. The former is what we actually refer to as the **creation** of an object, while the latter the **initialization** of the newly created object.

> [!TIP]
>
> Object creation is like getting something out of thin air, without specifying what it has. It provides an object that we can operate on. Object initialization is filling the content according to some specifications, making it actually usable.

	In our dictionary case, object creation is done by simply calling `dict`. However, in the context of classes, what Python actually operates on is not a plain dictionary, but a dictionary-like object with some other information. It would be a nightmare to go through the creation process manually by ourselves.
	
	Fortunately, Python handles creation automatically when it comes to classes. Therefore, we don't need to manually write something like `dispatch = dict()`. Instead, we only need to assume that the object has been created, and handle its initialization.

> [!TIP]
>
> Actually, we *are* allowed to manipulate the creation process. This is done through adding a correctly implemented function called `__new__` into the `class` block. We're not going to cover how we can write one ourselves as it could be overly complex. If you're interested, you may refer to materials on the Internet.

	This requires us to make some changes to our `create_tree` function. Instead of creating a dictionary and plugging key-value pairs into it, we take an already created empty object, and use the dot expression to initialize it.

```python
def initialize_tree(tree_dict, label, branches=None): # we call this initialize_tree now!
    if branches is None:
        branches = []
    tree_dict.label = label 		# this is like tree_dict['label'] = label
    tree_dict.branches = branches	# this is like tree_dict['branches'] = branches
    # we don't need to return anything now!
```

	Then we just need to fit it into the `Tree` class!

```python
class Tree:
    def initialize_tree(tree_dict, label, branches=None):
    	if branches is None:
        	branches = []
	    tree_dict.label = label
	    tree_dict.branches = branches
        
	def is_leaf(tree_dict):
        return not tree_dict.branches
    
    def print_me(tree_dict):
        ...
    
    def is_binary(tree_dict):
        ...
```

	I mentioned that Python automatically handles object creation. We can tell Python to create an object of a specific class (or type), we **call** the class name as if it were a function. For instance, to create a `Tree` object, we write `tr = Tree()`.
	
	Et voila! Now we can put it all together!

```python
class Tree:
	...

tr = Tree()
Tree.initialize_tree(tr, 1)
if Tree.is_leaf(tr):
	Tree.print_me(tr)
```

	Now all the functions have been neatly put away in the `Tree` class. The next goal is to get back to the promise of `tr.is_leaf()`. But surprise surprise! Creating an object by calling a class already enables us to do so! The designers of Python also see this urge, and have readily fulfilled the promise. If `tr` is created using `Tree()`, then `tr.is_leaf` is automatically interpreted as `Tree.is_leaf(tr)`! This is because in creating this object, Python secretly stores in it which class it belongs to. This makes it possible for all the magic that follows.
	
	With this knowledge, we can rewrite the code above as:

```python
class Tree:
    ...

tr = Tree()
tr.initialize_tree(1)
if tr.is_leaf():
    tr.print_me()
```

	This looks immediately cleaner. However, this is also where confusion kicks in. 
	
	At first glance, `is_leaf` takes one argument, yet in `tr.is_leaf()`, it looks like none is given. Similarly, `initialize_tree` takes 3 arguments, the first being `tree_dict`, but we're passing `1`, which should be the `label`. If these were normal functions, surely the code would be broken.
	
	This is because what we are actually invoking are not functions, but **bound methods**. While we will dig into the implementation details of methods later, we'll quickly cover the basic rules.
	
	Here is a quick mnemonic: if `obj` is created by calling a class `Foo`, and `f` is a function inside that class, then `obj.f(...)` is equivalent to `Foo.f(obj, ...)`.
	
	Python automatically passes the object on the left of the dot (`obj`) as the first argument, leaving us to provide **the rest of the arguments**. In functional programming terms (which you should be familiar with!), Python effectively **creates a partial function** where the first parameter is pre-filled with the object.
	
	I highly recommend you try a few examples in your Python interpreter to get the hang of this idea. We'll be relying on this syntax a lot in the rest of this article. So take your time and get your hands dirty before moving on.

> [!Tip]
>
> By now, we've actually solved our dilemma. Compare all previous examples and try to wrap your head around how our initial idea evolves.

	Now I assume you've played around with classes for some time. You may have noticed that each time we want to get a usable object, we would have to repeat code like this:

```python
tr1 = Tree()
tr1.initialize_tree(1)
tr2 = Tree()
tr2.initialize_tree(2)
tr3 = Tree()
tr3.initialize_tree('And more!', [tr1, tr2])
```

	This process clearly separates the logic of creation and initialization. However, it feels a little redundant since most of the time, object initialization follows immediately after object creation. Actually, Python allows us to merge these two processes into one single expression. 
	
	To tell Python to invoke a function immediately after object creation, we simply name the function to be invoked as `__init__`. In our `Tree` case, we want Python to call `initialize_tree` on the newly-created object, so we rename it to `__init__`. We call `__init__` the **initializer**.

> [!Tip]
>
> Functions surrounded by double underscores, like `__init__`, are called **dunder methods**, where *dunder* is abbreviated from **d**ouble **under**score. They're also referred to as **magic methods** because they're implicitly invoked on certain occasions (such as `__init__` being invoked after object creation).

> [!Tip]
>
> You may find some tutorials and other materials refer to `__init__` as the **constructor**. In many languages, the term constructors are used interchangeably with initializers. However, in Python, constructors actually refer to the `__new__` method, as has been mentioned before in a tip.
>
> But most of the time, we don't pay attention to this distinction. When someone says *constructor*, they probably means the `__init__` method unless they're talking about advanced Python features.

	As is often the case, initializers take arguments. On such occasions, we simply call the class (remember that classes can be called like functions!) with the desired arguments.

```python
class Tree:
	def __init__(tree_dict, label, branches=None): # name changed!
        ...
    ...

# the old way
# tr1 = Tree()
# tr1.initialize_tree(1)

# the concise way
tr2 = Tree(1)
```

	Put simply, if you were to call `obj = Foo(); obj.__init__(...)`, just write `obj = Foo(...)`. This is a convenient shorthand, and not every class must contain a `__init__` function. If Python doesn't find `__init__`, just as in our previous code snippets, it simply does nothing after creating the object.
	
	Now our `Tree` class looks like this:

```python
class Tree:
	def __init__(tree_dict, label, branches=None):
        if branches is None:
        	branches = []
	    tree_dict.label = label
	    tree_dict.branches = branches
        
	def is_leaf(tree_dict):
        return not tree_dict.branches
    
    def print_me(tree_dict):
        ...
    
    def is_binary(tree_dict):
        ...
```

	However, if you take our final `Tree` class to anyone familiar with Python, they are going to raise an eyebrow. Look at the functions sitting inside the class, and you may notice that all of them take `tree_dict`, the object to be operated on, as the first argument. And in fact, this is mostly the case. Therefore, the Python community has a convention to always name the first argument as `self`. With this little tweak, we can be quite satisfied with our `Tree` class now.

```python
class Tree: # rename all "tree_dict" to "self"!
	def __init__(self, label, branches=None):
        if branches is None:
        	branches = []
	    self.label = label
	    self.branches = branches
        
	def is_leaf(self):
        return not self.branches
    
    def print_me(self):
        ...
    
    def is_binary(self):
        ...
```

	Again, I recommend that you pause here to reflect on how classes save us from the dilemma. It may be beneficial to still think in terms of dispatch dictionaries and floating functions and then map concepts to the `class` syntax when writing code related with classes. It may also help to decode the `class` syntax to dictionaries and floating functions when reading code using classes.
	
	Before proceeding to the next section, do try writing some code with classes. It is something that takes some time to get familiar with even you're fully comfortable with the ideas behind it (which I hope you are!).

## More Dictionaries

 

## WTF is Inheritance?

This section first introduces inheritance via a problem (taken from previous lab/hw), tries to solve it in the dictionary setting, and walks through the inheritance syntax and behavior in Python. 

## Stop Joking, Python.

**Important: This section is beyond the scope of the course. You don't have to understand everything in this section.**

This section take a deep dive into MRO, attribute look-up mechanisms and descriptors. This is advanced knowledge and may be confusing.