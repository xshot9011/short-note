# Python Programming

## reducing memory using

```python
from random import random

class Point:
    def __init__(self, x):
        self.x = x


objects = []
for _ in range(1000000):
    r = random()  # The floating point numbers: 11% of memory.
    # Point objects in general: 30% of memory.
    point = Point(r)  # Adding an attribute to Point‘s dictionary: 55% of memory.
    objects.append(point)  # The list storing the Point objects: 4% of memory.
```

### Solution #1: Good bye, dictionary!

```python
from random import random

class Point:
    __slots__ = ["x"]  # <-- allowed attributes
    def __init__(self, x):
        self.x = x


objects = []
for _ in range(1000000):
    r = random()
    point = Point(r)
    objects.append(point)
```

ตั้งค่าแค่ตัวที่เป็น attr จริงๆ >> Python ไม่สามารถ dict สำหรับทุกๆ obj

>>The overhead for the dictionary is now gone, and memory use has reduced by 60%, from 207MB to 86MB. Not bad for one line of code!

### Solution #2: Get rid of objects

```python
from random import random

points = {
    "x": [],
    # "y": [],
    # "z": []
    # etc.
}

for _ in range(1000000):
    r = random()
    points["x"].append(r)
```

คือการทำงานบางอย่างเราก็ไม่จำเป็นต้องใช้ obj

>> Memory usage is now reduced to 30MB, down 85% from the original 206MB:

### Best solution

Bonus, even-better solution: Pandas instead of dict-of-lists