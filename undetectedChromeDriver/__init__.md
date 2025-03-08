# __init__ .py

When you import Chrome from undetectable_chromedriver package, You're actually importing __Chrome__ class from this file.

In `Chrome.__init__` there are some basic implementations for running bot, but there are also some arguments provided that you can pass value to them as given parameters when initialize `Chrome()` from this module for better configurations and less detection by anti-bot systems.

- `__init__`():

This is the constructor function for initialize an instance of Chrome class.

At the beginning of function, an instance of `Patcher` class initialize and store in __self.patcher__. The `Patcher` class is available at __patcher.py__ and we'll discuss about it later, but for summary, the patcher class will prepare chrome driver for execution.
