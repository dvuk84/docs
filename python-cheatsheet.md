* [PRINTING, USER INPUT AND ARGUMENTS](#PRINTING)
* [FILE HANDLING](#FILES)
* [NAMES, VARIABLES, CODE AND FUNCTIONS](#VARS)
* [CONDITIONS, LOOPS AND LISTS](#LOOPS)
* [TUPLES](#TUPLES)
* [DICTIONARIES](#DICT)
* [SETS](#SETS)
* [FUNCTIONS, MODULES, CLASSES AND OBJECTS](#OBJECTS)
* [INHERITANCE](#INHERITANCE)
* [BUILTIN MODULES](#BUILTIN)
* [EXIT CODE](#EXIT)
* [REFERENCE](#REFERENCE)

<a name="PRINTING"></a>
#### PRINTING, USER INPUT AND ARGUMENTS

```python
# argument
from sys import argv

first, second = argv
print("This is your arg: "), second
```

```python
# user input
prompt = ">"
print("What's your name?")
like = raw_input(prompt)
```

```python
# formating
formatter = "%r %r %r"
print formatter % ("one", "two", "three")
```

```python
# printing
print("hello", name, "bye")
print("hello" + name + "bye")
print("hello %s %d", % (name, age))	# python 2
print("hello {} {}".format(name, age))
print("hello" *5)
print("There are {0} days in {1}, {2} and {3}".format(31, "January", "March", "April"))
print("Number {0:2} squared is {1:4}".format(2, 2 ** 2))
print("""string split
         across
         multiple lines""")
```

<a name="FILES"></a>
#### FILE HANDLING

```
# file modes
r - read (default)
w - write
x - create new file
a - append
b - binary
t - text (default)
+ - update (read-write)
```

```python
# read from file
file = sample.txt
txt = open(file, 'r')
print(txt.read())
```

```python
# write to file
filename = sample.txt
txt = open(filename, 'rw')

text = "writing to a file"

# either
txt.truncate()
txt.write(text)

# or
print(name, file=filename)

txt.close()
```

```python
# copy file
from_file = "source.txt"
to_file = "target.txt"

in_file = open(from_file, 'r')
indata = in_file.read()         # read entire file as a string
indata = in_file.readline()     # read line by line

for line in in_file.lower():
    print(line, end='')

out_file = open(to_file, 'w')
out_file.write(indata)

out_file.close()
in_file.close()
```

```python
# print 5th line
def rewind(f):
    f.seek(5)

rewind("file.txt")
```

```python
# pickle
import pickle
imelda = ("More Mayhem", "Imelda May", (1, "Polling the Rug"))

with open("imelda.pickle", "wb") as pickle_file:
    pickle.dump(imelda, pickle_file)
```

```python
# shelve
import shelve

with shelve.open('Shelf.Test') as fruit:
    fruit['orange'] = "a sweet citrus fruit"
    fruit['apple'] = "good for making cider"
    
dict_key = fruit['orange']
description = fruit.get(dict_key)
print(description)
fruit.close()
```

<a name="VARS"></a>
#### NAMES, VARIABLES, CODE AND FUNCTIONS

```python
# global variables
global age
```

```python
# non-local variable
nonlocal tab_stop
```

```python
# integers and strings
name = "Variable"
print(name[0:6])
print(name[4:])
print(name[-1])
print(name[0:6:2])	# Vra
print("a" in name)	# True
```

```python
# multiple arguments
def print_two(*args):
    arg1, arg2 = args
    print("arg1: %r, arg2: %r") % (arg1, arg2)

print_one("One", "Two")
```

```python
# return value
def cheese_and_crackers(count, boxes):
    print("You have %d crackers in %d boxes") % (count, boxes)
    return count + boxes

howmany = cheese_and_crackers(20, 30)
```

```python
# protected function
_deal_card(frame):
```

<a name="LOOPS"></a>
#### CONDITIONS, LOOPS AND LISTS

```python
# if else
if one < two:
    print("do this")
elif two > one:
    print("do that instead")
else:
    print("don't know what to do")
```

```python
# for loop
my_list = ['one', 'two', 'three']
for i in my_list:
    print("element: %s") % i

for i in range(0, 6):
    print("adding %d to the list") % i
    my_list.append(i)
```

```python
# while loop
while i < 6:
    print("fourth element: %d") % i[3]
```

```python
# iterators
string = "1234567890"
for char in iter(string):
    print(char)

my_iterator = iter(string)
for i in range(0, len(string)):
    next_item = next(my_iterator)
    print(next_item)
```

```python
# range
decimals = range(0, 10)
my_range = decimals[::2]

print(list(range(0, 5, 2)))
```

```python
# traverse a list
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

<a name="TUPLES"></a>
#### TUPLES

```python
# tuple
t = "abc", "b", "c", 1975
t2 = ("a", "b", ("d", "e"))
print(t)
print(("abc", "b", "c", 1975))
print(t[0])				# abc
```

<a name="DICT"></a>
#### DICTIONARIES

```python
# dictionary
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

```python
# get element
input_string = my_dict.get(input("Enter: "), "Error: Element not found")
```

```python
# join() instead of concatenation with +
mylist = ["a", "b", "c"]
for i in mylist:
    new_string = ", ".join(mylist)
```

```python
# create a list of keys from a dictionary
ordered_keys = list(states.keys())
ordered_keys.sort()
```

```python
# convert dictionary to a tuple
f_tuple = tuple(states.items())
print(f_tuple)

for state in f_tuple:
    key, val = state
    print(key, val)
```

```python
# items() with key and value tuples
for key, val in fruit.items():
    print(key, val)
```

```python
# merging dictionaries
fruit = { "orange": "a sweet citrius fruit",
          "apple": "good for making cider" }

veg = {"cabbage": "every child's favourite",
       "sprouts": "mmmmm, lovely" }

veg.update(fruit)
```

```python
# copying instead of modying the current dictionary
both = fruit.copy()
both.update(veg)
```

<a name="SETS"></a>
#### SETS

```python
# set
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

```python
# union and intersection
print(farm_animals.union(wild_animals))
print(farm_animals.intersection(wild_animals))
```

<a name="OBJECTS"></a>
#### FUNCTIONS, MODULES, CLASSES AND OBJECTS

```python
# import
from turtle import forward, done
from turtle import *
```

```python
# my_module.py
def apple():
    print("I AM APPLES")

import my_module
my_module.apple()
```

##### class
```python
# class
class MyClass(object):
    def __init__(self):
        self.tangerine = "And now a thousand years between"

    def apple(self):
        print("I AM APPLES")
```

```python
# another example of a class
class Song(object):
    def __init__(self, lyrics):
        self.lyrics = lyrics

    def sing_me_a_song(self):
        for line in self.lyrics:
            print(line)
happy_bday = Song(["Happy birthday to you"])
happy_bday.sing_me_a_song()
```

```python
# arguments
def print(*args, sep=' ', end ='\n', file=none, flush=None):
    for arg in args:
        print(arg)

print("first", "second", 3, 4, "spam", sep=':')
```

<a name="INHERITANCE"></a>
#### INHERITANCE

```python
# implicit inheritance
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

```python
# super
TBA
```

<a name="BUILTIN"></a>
#### BUILTIN MODULES

```python
# webbrowser
import webbrowser
webbrowser.open("http://google.com")
```

```python
# time
import time
print(time.gmtime(0))
print(time.localtime())
print(time.time())
print(time.localtime().tm_year)
```

```python
# timer
from time import time as my_timer
import random
wait_time = random.randint(1, 6)
start_time = my_timer()
end_time = my_timer()
print(time.strftime("%X", time.localtime(start_time)))
```

```python
# timezone
import time
print(time.strftime('%c', time.gmtime(0))
print(time.tzname[0], time.timezone)
print(time.strftime('%Y-%m-%d', time.localtime())
```

```python
# datetime
import datetime
print(datetime.datetime.today())
print(datetime.datetime.now())
```

```python
# user time
from time import perf_counter
start_time = perf_counter()
end_time = perf_counter()
```

```python
# ?
from time import monotonic
start_time = monotonic()
end_time = monotonic()
```

```python
# cpu time
from time import process_time
```

```python
# tkinter
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

<a name="EXIT"></a>
#### EXIT CODE

```python
# exit code
from sys import exit
exit(0)
```

<a name="REFERENCE"></a>
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
