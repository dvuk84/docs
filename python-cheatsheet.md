#### PRINTING, USER INPUT AND ARGUMENTS

```python
from sys import argv
```

##### argument
```python
first, second = argv
print("This is your arg: "), second
```

##### user input
```python
prompt = ">"
print("What's your name?")
like = raw_input(prompt)
```

##### format
```python
formatter = "%r %r %r"
print formatter % ("one", "two", "three")
```

##### printing
```python
print("hello", name, "bye")
print("hello" + name + "bye")
print("hello %s %d", % (name, age))	# python 2
print("hello {} {}".format(name, age))
print("hello" *5)
print("There are {0} days in {1}, {2} and {3}".format(31, "January", "March", "April"))
print("N:w
umber {0:2} squared is {1:4}".format(2, 2 ** 2))
print("""string split
across
multiple lines""")
```

#### FILE HANDLING

##### r (default), w, x, a, b, t (default), +

##### read file
```python
file = sample.txt
txt = open(file, 'w')
print(txt.read())
```

##### write to file
```python
txt.truncate()
txt.write("writing to a file\n")
txt.close()

```
```python
print(name, file=filename.txt)
```

##### copy file
```python
from_file = "source.txt"
to_file = "target.txt"
```

```python
in_file = open(from_file, 'r')
indata = in_file.read()
indata = in_file.readline()
```

```python
for line in in_file.lower():
    print(line, end='')
```

```python
out_file = open(to_file, 'w')
out_file.write(indata)
```

```python
out_file.close()
in_file.close()
```

##### print 5th line
```python
def rewind(f):
    f.seek(5)

rewind("file.txt")
```

##### binary
```python
byte([age])
age.to_bytes(2, 'big')
e = int.from_bytes(bin_file(2), 'big'))
```

##### pickle
```python
import pickle
imelda = ("More Mayhem", "Imelda May", (1, "Polling the Rug"))

with open("imelda.pickle", "wb") as pickle_file:
    pickle.dump(imelda, pickle_file)
```

##### shelve
```python
import shelve

with shelve.open('Shelf.Test') as fruit:
    fruit['orange'] = "a sweet citrus fruit"
    fruit['apple'] = "good for making cider"
    
dict_key = fruit['orange']
description = fruit.get(dict_key)
print(description)
fruit.close()
```

#### NAMES, VARIABLES, CODE AND FUNCTIONS

##### global variable
```python
global age
```

##### non-local variable
```python
nonlocal tab_stop
```

##### integers and strings
```python
name = "Variable"
print(name[0:6])
print(name[4:])
print(name[-1])
print(name[0:6:2])	# Vra
print("a" in name)	# True
```


##### multiple arguments
```python
def print_two(*args):
    arg1, arg2 = args
    print("arg1: %r, arg2: %r") % (arg1, arg2)

print_one("One", "Two")
```

##### return value
```python
def cheese_and_crackers(count, boxes):
    print("You have %d crackers in %d boxes") % (count, boxes)
    return count + boxes

howmany = cheese_and_crackers(20, 30)
```

##### protected functions
```python
_deal_card(frame):
```

#### CONDITIONS, LOOPS AND LISTS

##### if else
```python
if one < two:
    print("do this")
elif two > one:
    print("do that instead")
else:
    print("don't know what to do")
```

##### fop loop and lists
```python
my_list = ['one', 'two', 'three']
for i in my_list:
    print("element: %s") % i

for i in range(0, 6):
    print("adding %d to the list") % i
    my_list.append(i)
```

##### while loop
```python
while i < 6:
    print("fourth element: %d") % i[3]
```

##### iterators
```python
string = "1234567890"
for char in iter(string):
    print(char)

my_iterator = iter(string)
for i in range(0, len(string)):
    next_item = next(my_iterator)
    print(next_item)
```

##### range
```python
decimals = range(0, 10)
my_range = decimals[::2]

print(list(range(0, 5, 2)))
```

##### traverse a list
```python
myvar = "This is a sentence"
myarray = ["one", "two"]
splitvar = myvar.split(' ')

while len(splitvar) != 10:
    next_one = myarray.pop()
    splitvar.append(next_one)

print(splitvar[1])
print(" ".join(splitvar))
print("#".join(splitvar[2:4])
```

#### TUPLES

##### tuples are not immutable
```python
t = "abc", "b", "c", 1975
t2 = ("a", "b", ("d", "e"))
print(t)
print(("abc", "b", "c", 1975))
print(t[0])				# abc
```

#### DICTIONARIES

```python
my_dict = {"name": "Mark", "age": 34}
print(my_dict["name"])

my_dict["age"] = 44
print(my_dict["age"])

my_dict[1]="wow"
print(my_dict[1])

states = [
    "Oregon": "OR",
    "Florida": "FL",
    "Michigan": "MI"
]
print(states["OR"])
```

##### get element
```python
input_string = my_dict.get(input("Enter: "), "Error: Element not found")
```

##### join() instead of concatenation with +
```python
mylist = ["a", "b", "c"]
for i in mylist:
    new_string = ", ".join(mylist)
```

##### create a list of keys from a dictionary
```python
ordered_keys = list(states.keys())
ordered_keys.sort()
```

##### convert dictionary to a tuple
```python
f_tuple = tuple(states.items())
print(f_tuple)

for state in f_tuple:
    key, val = state
    print(key, val)
```

##### items() with key and value tuples
```python
for key, val in fruit.items():
    print(key, val)
```

##### merging dictionaries
```python
fruit = { "orange": "a sweet citrius fruit",
          "apple": "good for making cider" }

veg = {"cabbage": "every child's favourite",
       "sprouts": "mmmmm, lovely" }

veg.update(fruit)
```

##### copying instead of modying the current dictionary
```python
both = fruit.copy()
both.update(veg)
```

#### SETS

```python
farm_animals = { "sheep", "cow", "hen" }
wild_animals = set(["lion", "tiger", "sheep"])
empty_set = set()

for animal in farm_animals:
    print(animal)

for animal in wild_animals:
    print(animal)

farm_animals.add("horse")
wild_animals.add("horse")
```

##### union and intersection
```python
print(farm_animals.union(wild_animals))
print(farm_animals.intersection(wild_animals))
```

#### FUNCTIONS, MODULES, CLASSES AND OBJECTS

##### import
```python
from turtle import forward, done
from turtle import *
```

##### this goes in my_module.py
```python
def apple():
    print("I AM APPLES")

import my_module
my_module.apple()
```

##### class
```python
class MyClass(object):
    def __init__(self):
        self.tangerine = "And now a thousand years between"

    def apple(self):
        print("I AM APPLES")
```

##### another example of a class
```python
class Song(object):
    def __init__(self, lyrics):
        self.lyrics = lyrics

    def sing_me_a_song(self):
        for line in self.lyrics:
            print(line)
happy_bday = Song(["Happy birthday to you"])
happy_bday.sing_me_a_song()
```

##### arguments
```python
def print(*args, sep=' ', end ='\n', file=none, flush=None):
    for arg in args:
        print(arg)

print("first", "second", 3, 4, "spam", sep=':')
```

#### INHERITANCE

##### implicit inheritance
```python
class Parent(object):
    def implicit(self):
      print("PARENT implicit()")

class Child(Parent):
    pass

dad = Parent()
son = Child()
dad.implicit()
son.implicit()

# explicit override
class Child(Parent):
    def override(self):
      print("CHILD override()")
```

##### super
```python
TBA
```

#### BUILTIN MODULES

##### webbrowser
```python
import webbrowser
webbrowser.open("http://google.com")
```

##### time
```python
import time
print(time.gmtime(0))
print(time.localtime())
print(time.time())
print(time.localtime().tm_year)
```

##### timer
```python
from time import time as my_timer
import random
wait_time = random.randint(1, 6)
start_time = my_timer()
end_time = my_timer()
print(time.strftime("%X", time.localtime(start_time)))
```

##### timezone
```python
import time
print(time.strftime('%c', time.gmtime(0))
print(time.tzname[0], time.timezone)
print(time.strftime('%Y-%m-%d', time.localtime())
```

##### datetime
```python
import datetime
print(datetime.datetime.today())
print(datetime.datetime.now())
```

##### user time
```python
from time import perf_counter
start_time = perf_counter()
end_time = perf_counter()
```

##### ?
```python
from time import monotonic
start_time = monotonic()
end_time = monotonic()
```

##### cpu time
```python
from time import process_time
```

##### tkinter
```python
import tkinter
print(tkinter,TkVersion)
tkinter.text()
mainWindow = tkinter.Tk()
mainWindow.title("hello")
mainWindow.geometry('640x480')

label = tkinter.Label(mainWindow, text="hello")
label.pack(side='top')
canvas = tkinter.Canvas(mainWindow, relief='raised', borderwidth=1)
canvas.pack(side='left')

mainWindow.mainloop()
```

#### EXIT CODE

```python
from sys import exit
exit(0)
```

#### REFERENCE

##### keywords
```
• and
• del
• from
• not
• while
• as
• elif
• global
• or
• with
• assert
• else
• if
• pass
• yield
• break
• except
• import
• print
• class
• exec
• in
• raise
• continue
• finally
• is
• return
• def
• for
• lambda
• try
```

##### data types
```
• True
• False
• None
• strings
• numbers
• floats
• lists
```

##### string escape sequences
```
• \\
• \'
• \"
• \a
• \b
• \f
• \n
• \r
• \t
• \v
```

##### string formats
```
• %d
• %i
• %o
• %u
• %x
• %X
• %e
• %E
• %f
• %F
• %g
• %G
• %c
• %r
• %s
• %%
```

##### operators
```
• +
• -
• *
• **
• /
• //
• %
• <
• >
• <=
• >=
• ==
• !=
• <>
• ( )
• [ ]
• { }
• @
• ,
• :
• .
• =
• ;
• +=
• - =
• *=
• /=
• //=
• %=
• **=
```