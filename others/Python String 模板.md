```python
>>> import string
>>> greeting = string.Template("Hello, $name, good $time!")
>>>
>>> greeting.substitute(name="OpenSource.com", time="afternoon")
'Hello, OpenSource.com, good afternoon!'

```