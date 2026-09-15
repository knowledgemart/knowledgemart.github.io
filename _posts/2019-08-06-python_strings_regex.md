---
title: Python regular expressions and string manipulation
date: 2019-08-06 00:00:00 +0100
tags: [Python]
---

Text processing and manipulation are essential steps in data cleaning and preparation. Machine learning models that rely on text — such as _sentiment analysis, language translation, spam filtering,_ or _data extraction from web pages_ — depend heavily on these techniques. Regular expressions make these operations faster and more efficient, enabling complex text transformations with minimal code.  
  
## Basic String Methods

Python treats any sequence of characters enclosed in quotes as a **string object**. Strings can be defined using either single quotes (`'example'`) or double quotes (`"example"`). Both are equivalent, but when a quote character appears inside a string, you need to alternate the delimiters:

```python
print('This isn't going to work')
```
```
  File "<input>", line 1
    print('This isn't going to work!')
                    ^
SyntaxError: invalid syntax
```

```python
print("But this isn't going to cause any problems!")
```
```
But this isn't going to cause any problems!
```

Use triple quotes (`'''` or `"""`) to create multi-line strings:

```python
print('''This is a
multi-line
string''')
```
```
This is a
multi-line
string
```

### Length of a string

The built-in `len()` function returns the number of items in a collection. For strings, it gives the number of characters. You can also use `len()` function on other collection types like list, dict, tuple etc. 

```python
my_string = 'I wonder how many characters are in this string...'
len(my_string)
```
```
50
```

### Concatenation 
Python supports several ways to concatenate (combine) strings.

1. Using the `+` operator.

    ```python
    str_1 = 'some '
    str_2 = 'random '
    str_3 = 'words'

    print(str_1 + str_2 + str_3)
    ```
    ```
    some random words
    ```

2. Using the `+=` operator to append:

    ```python
    str_1 = 'some '
    str_2 = 'random '
    str_3 = 'words'

    str_1 += str_2
    str_1 += str_3

    print(str_1)
    ```
    ```
    some random words
    ```

3. Using the `join()` method — useful when joining multiple strings from an iterable:

    ```python
    str_1 = 'some '
    str_2 = 'random '
    str_3 = 'words'

    print(''.join([str_1, str_2, str_3]))
    ```
    ```
    some random words
    ```

4. Using f-strings (Python 3.6+):

    ```python
    str_1 = 'some '
    str_2 = 'random '
    str_3 = 'words'

    print(f'{str_1}{str_2}{str_3}')
    ```
    ```
    some random words
    ```

5. Using the `format()` method:

    ```python
    str_1 = 'some '
    str_2 = 'random '
    str_3 = 'words'

    print('{}{}{}'.format(str_1, str_2, str_3))
    ```
    ```
    some random words
    ```

### Slicing
Slicing extracts parts of a string using the syntax: `string[start:stop:step]`

- `start` - index of the first character to include (default: 0)
- `stop` — index of the first character to exclude (default: string’s length)
- `step` - number of characters to skip after each selection (default: 1)

Negative indices count from the end of the string. For example, reversing a string is done by using `[::-1]`.

```python
my_string = 'fantastic'
print(my_string[0])      # first character
print(my_string[1:4])    # some middle characters
print(my_string[-1])     # last character
print(my_string[::2])    # every second character
print(my_string[::-1])   # reversed string
```
```
f
ant
c
fnatc
citsatnaf
```

### String capitalization
Python provides several methods for changing the capitalization of strings:

1. `upper()` - converts all characters to uppercase

2. `lower()` - converts all characters to lowercase

3. `capitalize()` - capitalizes the first character only

4. `title()` - capitalizes the first letter of each word

5. `swapcase()` - inverts the case of all letters

```python
my_string = 'whAT HaPPened to THESE leTTerS?'
print(my_string.upper())
print(my_string.lower())
print(my_string.capitalize())
print(my_string.title())
print(my_string.swapcase())
```
```
WHAT HAPPENED TO THESE LETTERS?
what happened to these letters?
What happened to these letters?
What Happened To These Letters?
WHat hAppENED TO these LEttERs?
```

### Split methods
Python offers multiple ways to split strings into smaller parts:

1. `split()` – divides a string into a list of substrings using a specified delimiter. If no delimiter is given, it defaults to any whitespace character (spaces, tabs, newlines).
2. `rsplit()` – behaves like `split()` but starts splitting from the right.
3. `splitlines()` – splits a string at newline characters, returning a list of lines.
4. `partition()` – splits a string at the first occurrence of a specified separator and returns a **tuple** containing:
   - the part before the separator,  
   - the separator itself, and  
   - the part after.

    ```python
    my_string = '''This string 
    should be split 
    into separate parts'''
    
    print(my_string.split())
    print(my_string.rsplit())
    print(my_string.splitlines())
    print(my_string.partition(' '))
    ```
    ```
    ['This', 'string', 'should', 'be', 'split', 'into', 'separate', 'parts']
    ['This', 'string', 'should', 'be', 'split', 'into', 'separate', 'parts']
    ['This string ', 'should be split ', 'into separate parts']
    ('This', ' ', 'string \nshould be split \ninto separate parts')
    ```

5. `re.split()` -– splits a string using a **regular expression** as the delimiter. This requires importing the `re` module. Examples will appear later in the tutorial.

### Remove whitespace
To remove spaces, tabs, or newlines, Python provides several methods:

1. `strip()` - removes both leading and trailing whitespace
2. `lstrip()` - removes leading (left) whitespace only
3. `rstrip()` - removes trailing (right) whitespace only
4. `replace(' ', '')` – removes all spaces by replacing them with an empty string
5. `' '.join(string.split())` – splits and rejoins text to normalize spacing (=> single space only)

```python
my_string = '   There is way too   much  whitespace here.     '

print(my_string.strip())
print(my_string.rstrip())
print(my_string.lstrip())
print(my_string.replace(' ', ''))
print(' '.join(my_string.split()))
```
```
There is way too   much  whitespace here.
   There is way too   much  whitespace here.
There is way too   much  whitespace here.     
Thereiswaytoomuchwhitespacehere.
There is way too much whitespace here.
```

### Replace methods {#replace-methods}

There are several ways to replace strings in Python.

1. The `replace()` method replaces all occurrences of a specified substring with another substring.

    ```python
    my_string = 'Mr Heckles has a cat. Or he could have a cat. You owe him a cat.'
    # Any Friends fans here? ;)

    print(my_string.replace('cat', 'dog'))
    ```
    ```
    Mr Heckles has a dog. Or he could have a dog. You owe him a dog.
    ```

2. The `translate()` method of string can be used to replace multiple characters in a string. This method uses a translation table, which is a mapping of characters to their replacement characters, to perform the replacements. It's important to note that this method only works for **single-character replacements**.

    ```python
    my_string = "Let's replace some vowels."

    translation_table = str.maketrans('eo', 'xz')

    print(my_string.translate(translation_table))
    ```
    ```
    Lxt's rxplacx szmx vzwxls.
    ```

3. The `join()` method along with list comprehension can be used to replace substrings. The method takes a list or iterable of strings as an argument, and concatenates them using the calling string as the separator. It's important to note that this method only works for replacing substrings that are whole words.

    ```python
    my_string = 'Today is a good day and I feel really good too.'

    words = ['great' if word == 'good' else word for word in my_string.split()]
    my_string = ' '.join(words)
    print(my_string)
    ```
    ```
    Today is a great day and I feel really great too.
    ```

4. The `re.sub()` function from the re module can also be used to replace substrings using regular expressions. Examples will follow later in this tutorial.

<br>
### Chain string methods {#chain-string-methods}
In Python, it is possible to chain multiple string methods together in a single line of code. This is done by calling one string method, and then immediately calling another method on the result

```python
string = 'Hello, world!'
string = string.replace('world', 'python').title()
print(string)
```
```
Hello, Python!
```

It is important to note that the order of the methods being called is important, as the output of one method is used as the input for the next method.

It is also important to note that the chain of methods will be called in the order they are written, i.e. the leftmost method will be called first.

<br>
#### Find the position of a substring {#find-substring-position}

In Python, there are a few different built-in string methods that can be used to find the position of a substring within a larger string.

1. `str.find(sub)` - returns the **lowest index** in the string where substring `sub` is found. If the substring is not found, it returns `-1`.

2. `str.index(sub)` - similar to `find()`, but raises a `ValueError` exception if the substring is not found.

3. `str.rfind(sub)` - returns the **highest index** in the string where substring `sub` is found. If the substring is not found, it returns `-1`.

4. `str.rindex(sub)` - similar to `rfind()`, but raises a `ValueError` exception if the substring is not found.

Note that the methods `find()`, `rfind()`, `index()`, and `rindex()` search for the substring from left to right.

`find()` method returns the starting position of the first occurence of a given string. Optionally we can include `start` and `end` parameters (where starting position is inclusive but ending position is not).

```python
my_string = 'Mr Heckles has a cat. Or he could have a cat. You owe him a cat.'

print(my_string.find('cat'))
print(my_string.find('dog'))
print(my_string.index('cat'))
```
```
17
-1
17
```
```python
print(my_string.index('dog'))
```
```
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
Cell In[50], line 1
----> 1 print(my_string.index('dog'))

ValueError: substring not found
```
```python
print(my_string.rfind('cat'))
print(my_string.rfind('dog'))
print(my_string.rindex('cat'))
```
```
60
-1
60
```
```python
print(my_string.rindex('dog'))
```
```
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
Cell In[52], line 1
----> 1 print(my_string.rindex('dog'))

ValueError: substring not found
```

<br>
### Count the occurrences of a substring {#count-substr-occurrences}

`str.count(sub)` method returns the number of non-overlapping occurrences of substring `sub` in the string `str`.

```python
my_string = 'I love python, I cannot imagine my life without python. You can do so many amazing things with python.'

print(my_string.count('python'))
```
```
3
```

<br>
You can use the `str.replace(old, new[, count])` method with optional `count` argument to specify how many occurencies (at most) of a substring should be replaced.

```python
my_string = 'Mr Heckles has a cat. Or he could have a cat. You owe him a cat.'

print(my_string.replace('cat', 'dog'))
print(my_string.replace('cat', 'dog', 1))
```
```
Mr Heckles has a dog. Or he could have a dog. You owe him a dog.
Mr Heckles has a dog. Or he could have a cat. You owe him a cat.
```

Replace methods can be especially useful when preparing data for sentiment analysis algorithms. Some algorithms may not be able to understand negations. For example, a movie review might say the movie was _not good_. Sentiment analysis models splits the whole review into separate words or bags of words, ignore the _not_ part and conclude from the word _good_ that the sentiment of the review is positive. This issue could be avoided by replacing words preceded by _not_ with their antonyms.

<br>
### Positional string formatting {#positional-string-formatting}

Positional string formatting in Python is a method of placing placeholders in a string and then replacing them with values at runtime. The placeholders are denoted by curly braces `{}` and the values that replace them are passed as arguments to the `format()` method. The values are inserted into the placeholders in the order they are passed, with the first value replacing the first placeholder, the second value replacing the second placeholder, and so on.

```python
my_string = 'Mr Heckles has a {}. Or he could have a {}. You owe him a {}.'
print(my_string.format('cat', 'cat', 'cat'))
print(my_string.format('cat', 'dog', 'fish'))
```
```
Mr Heckles has a cat. Or he could have a cat. You owe him a cat.
Mr Heckles has a cat. Or he could have a dog. You owe him a fish.
```

<br>
It's also possible to reference the values by the position of the argument passed to the `format` method.

```python
my_string = 'Mr Heckles has a {1}. Or he could have a {2}. You owe him a {0}.'
print(my_string.format('cat', 'dog', 'fish'))
```
```
Mr Heckles has a dog. Or he could have a fish. You owe him a cat.
```
```python
my_string = 'Mr Heckles has a {0}. Or he could have a {0}. You owe him a {0}.'
print(my_string.format('cat', 'dog', 'fish'))
```
```
Mr Heckles has a cat. Or he could have a cat. You owe him a cat.
```

<br>
Placeholders can be named.

```python
my_string = '{owner} has a {animal}. Or he could have a {animal}. You owe him a {animal}.'
print(my_string.format(owner='Mr Heckles', animal='cat'))
```
```
Mr Heckles has a cat. Or he could have a cat. You owe him a cat.
```

<br>
It's possible to use both positional and named placeholders in the same `format()` method call.

```python
my_string = '{0} has a {animal}. Or he could have a {animal}. You owe him a {animal}.'
print(my_string.format('Mr Heckles', animal='cat'))
```
```
Mr Heckles has a cat. Or he could have a cat. You owe him a cat.
```

<br>
Positional string formatting in Python can use formatted placeholders, which allow you to specify the format of the values that replace the placeholders. Formatted placeholders are denoted by curly braces `{}` and a colon `:` followed by a format specifier, for example `{name:.2f}`.

```python
my_string = 'There is {chance:.2f}% chance that Mr Heckles had a cat.'
print(my_string.format(chance=0.0512349))
```
```
There is 0.05% chance that Mr Heckles had a cat.
```
```python
from datetime import datetime

print("Today's date: {}".format(datetime.now()))
print("Today's date: {:%Y-%m-%d}".format(datetime.now()))
```
```
Today's date: 2019-08-06 16:25:23.892789
Today's date: 2019-08-06
```

You can also use some standard Python format codes like `:d` for integers, `:f` for floating point numbers, `:.2f` for floating point numbers with two decimal places, `:s` for strings and so on.

These formatted placeholders provide more control over the appearance of the final string, and make it easy to ensure consistency in the formatting of the values.

<br>
You can use a dictionary to replace placeholders with specific values.

```python
my_dict = {
    'animal': 'cat',
    'owner': 'Mr Heckles'
}

my_string = '{d[owner]} has a {d[animal]}. Or he could have a {d[animal]}. You owe him a {d[animal]}.'

print(my_string.format(d=my_dict))
```
```
Mr Heckles has a cat. Or he could have a cat. You owe him a cat.
```

<br>
Python f-strings (formatted string literals) is a feature introduced in Python 3.6 that allows you to embed expressions inside string literals using `{}` (curly braces) and prefixing the string with the letter `'f'`. The expressions inside the curly braces are evaluated at runtime and the results are inserted into the string.

F-strings are considered as an alternative and more readable way of formatting strings, as the expressions are evaluated directly inside the string, instead of being passed as arguments to a separate method.

```python
owner = 'Mr Heckles'
animal = 'cat'

print(f'{owner} has a {animal}. Or he could have a {animal}. You owe him a {animal}.')
```
```
Mr Heckles has a cat. Or he could have a cat. You owe him a cat.
```

<br>
F-strings can also include formatted placeholders.

```python
pi = 3.14159

print(f'The value of pi is approximately {pi:.2f}.')
```
```
The value of pi is approximately 3.14.
```

In this example, the expression inside the curly braces `{pi:.2f}` is evaluated at runtime and the result is formatted to 2 decimal places and inserted into the string.

<br>
You can use dictionaries with f-strings as well.

```python
my_dict = {
    'animal': 'cat',
    'owner': 'Mr Heckles'
}

print(f"{my_dict['owner']} has a {my_dict['animal']}. "
      f"Or he could have a {my_dict['animal']}. "
      f"You owe him a {my_dict['animal']}.")
```
```
Mr Heckles has a cat. Or he could have a cat. You owe him a cat.
```

<br>
F-strings are more readable and efficient than the traditional string formatting methods, as they allow you to embed expressions directly inside the string, and the expressions are evaluated at runtime, which eliminates the need for a separate method call. They are also more flexible, as they allow you to include any valid Python expressions inside the curly braces.

```python
x = 10
y = 4

print(f'{x} times {y} equals {x*y}')
```
```
10 times 4 equals 40
```

You can even call python functions inside curly braces.

```python
def multiply(a, b):
    return a*b

x = 10
y = 4

print(f'{x} times {y} equals {multiply(x, y)}')
```
```
10 times 4 equals 40
```

<br>
## Regular Expressions {#regular-expressions}

Regular expressions (often shortened to "regex" or "regexp") are a powerful tool for matching patterns in strings. They are strings containing a combination of normal characters and special metacharacters used to describe patterns for finding text within a larger text. Potential use includes finding and replacing text, string validation or renaming a bunch of files. Python provides the `re` package for handling regex.

### Common methods {#common-methods}

```python
import re
```

<br>
The `search(pattern, string)` method searches for the first occurrence of the pattern in the given string.

```python
text = 'Mr Heckles has a cat. Or he could have a cat. You owe him a cat.'

print(re.search(r'cat', text))
```
```
<re.Match object; span=(17, 20), match='cat'>
```

<br>
The `findall(pattern, string)` method finds all occurrences of the pattern in the given string and returns them as a list.

```python
text = 'Mr Heckles has a cat. Or he could have a cat. You owe him a cat.'

print(re.findall(r'cat', text))
```
```
['cat', 'cat', 'cat']
```

<br>
The `split(pattern, string)` method splits the given string into a list at each occurrence of the pattern.

```python
text = 'Mr Heckles has a cat! Or he could have a cat! You owe him a cat!'

print(re.split(r'!', text))
```
```
['Mr Heckles has a cat', ' Or he could have a cat', ' You owe him a cat', '']
```

<br>
The `sub(pattern, repl, string)` method replaces all occurrences of the pattern in the given string with the specified replacement string.

```python
text = 'Mr Heckles has a cat! Or he could have a cat! You owe him a cat!'

print(re.sub(r'!', '...', text))
```
```
Mr Heckles has a cat... Or he could have a cat... You owe him a cat...
```

<br>
### Metacharacters {#metacharacters}

`\d` matches any digit:

```python
tweets = '10 tweets by @user1. 15 tweets by @user624. 99 tweets by @userN'

print(re.findall(r'\d', tweets))
```
```
['1', '0', '1', '1', '5', '6', '2', '4', '9', '9']
```

<br>
`\D` matches a non-digit:

```python
tweets = '10 tweets by @user1. 15 tweets by @user624. 99 tweets by @userN'

print(re.findall(r'\D', tweets))
```
```
[' ', 't', 'w', 'e', 'e', 't', 's', ' ', 'b', 'y', ' ', '@', 'u', 's', 'e', 'r', '.', ' ', ' ', 't', 'w', 'e', 'e', 't', 's', ' ', 'b', 'y', ' ', '@', 'u', 's', 'e', 'r', '.', ' ', ' ', 't', 'w', 'e', 'e', 't', 's', ' ', 'b', 'y', ' ', '@', 'u', 's', 'e', 'r', 'N']
```

<br>
_Example: search for usernames that start with the word `user` and are followed by at least one digit._

```python
tweets = '10 tweets by @user1. 15 tweets by @user624. 99 tweets by @userN'

print(re.findall(r'user\d', tweets))
```
```
['user1', 'user6']
```

<br>
`\w` matches any word character (alphanumeric and underscores). 

_Example: find every occurence of the string `user` followed by a word character._

```python
tweets = '10 tweets by @user1. 15 tweets by @user624. 99 tweets by @userN'

print(re.findall(r'user\w', tweets))
```
```
['user1', 'user6', 'userN']
```

<br>
`\W` matches any non-word (special) character:

```python
tweets = '10 tweets by @user1. 15 tweets by @user624. 99 tweets by @userN'

print(re.findall(r'\W', tweets))
```
```
[' ', ' ', ' ', '@', '.', ' ', ' ', ' ', ' ', '@', '.', ' ', ' ', ' ', ' ', '@']
```

<br>
`\s` matches any whitespace characters (i.e. spaces, tabs, linebreaks).

_Example: find every occurence of the string `tweets` preceded by a whitespace and two digits._

```python
tweets = '10 tweets by @user1. 15 tweets by @user624. 99 tweets by @userN'

print(re.findall(r'\d\d\stweets', tweets))
```
```
['10 tweets', '15 tweets', '99 tweets']
```

<br>
`\S` matches a non-whitespace.

```python
tweets = '10 tweets by @user1. 15 tweets by @user624. 99 tweets by @userN'

print(re.findall(r'user\S', tweets))
```
```
['user1', 'user6', 'userN']
```

<br>
_Example: replace every occurence of the string `by` with a colon `:`._

```python
tweets = '10 tweets by @user1. 15 tweets by @user624. 99 tweets by @userN'

print(re.sub(r'\sby', ':', tweets))
```
```
10 tweets: @user1. 15 tweets: @user624. 99 tweets: @userN
```

<br>
### Quantifiers {#quantifiers}

Let's see an example first. 

```python
text = 'password9876'

print(re.search(r'\w\w\w\w\w\w\w\w\d\d\d\d', text))
```
```
<re.Match object; span=(0, 12), match='password9876'>
```

<br>
The `{n}` quantifier can be used to tell the regex engine how many times exactly a character on its left should be matched. The above example can be rewritten as follows.

```python
text = 'password9876'

print(re.search(r'\w{8}\d{4}', text))
```
```
<re.Match object; span=(0, 12), match='password9876'>
```

<br>
The `+` quantifier is used to match **one or more occurrences** of the preceding character (or group).

```python
text = 'password9876, password 1, password123, password'

print(re.findall(r'password\d+', text))
```
```
['password9876', 'password123']
```

<br>
The `*` quantifier is used to match **zero or more occurrences** of the preceding character (or group).

```python
text = 'password9876, password 1, password123, password'

print(re.findall(r'password\d*', text))
```
```
['password9876', 'password', 'password123', 'password']
```

<br>
The `?` quantifier is used to match **zero or one occurrence** of the preceding character (or group).

```python
text = 'Center is American spelling. Centre is British'

print(re.findall(r'Cente?re?', text))
```
```
['Center', 'Centre']
```

<br>
The `{n,m}` quantifier is used to match a specific number of occurrences of the preceding character (or group) with `n` being the minimum number of occurrences and `m` being the maximum number of occurrences.

```python
text = 'password9876, password11, password123, password'

print(re.findall(r'password\d{3,4}', text))
```
```
['password9876', 'password123']
```

If you want to match **exactly** `n` occurrences, you can use the notation `{n}`.

```python
text = 'password9876, password11, password123, password'

print(re.findall(r'password\d{4}', text))
```
```
['password9876']
```

If you want to match **at least** `n` occurrences, you can use the notation `{n,}`.

```python
phone_numbers = '123-123-1234, 1-1-100, 999-456-123456'

print(re.findall(r'\d{3}-\d{3}-\d{4,}', phone_numbers))
```
```
['123-123-1234', '999-456-123456']
```

<br>
The dot `.` is a special character that represents any single character except for a newline. It is often used as a wildcard to match any character at a particular position in a string.

If you want to match a literal dot, you need to escape it like `\.`

```python
text = 'https://klaudiawolinska.github.io/'

print(re.findall(r'http.*', text))
```
```
['https://klaudiawolinska.github.io/']
```

```python
text = 'https://klaudiawolinska.github.io/'

print(re.findall(r'http\.*', text))
```
```
['http']
```

```python
text = "This is some text. It has 3 sentences. Let's split them."

print(re.split(r'\.\s', text))
```
```
['This is some text', 'It has 3 sentences', "Let's split them."]
```

<br>
### search() vs match() {#search-match}

The `search()` method searches for a match of a pattern in a string, while the `match()` method only matches patterns at the beginning of a string. The `search()` method is more flexible, as it can find matches anywhere in the string, whereas `match()` is more restrictive, as it only looks at the beginning.

```python
text = 'Center is American spelling. Centre is British'

print(re.search(r'Cente?re?', text))
print(re.match(r'Cente?re?', text))
```
```
<re.Match object; span=(0, 6), match='Center'>
<re.Match object; span=(0, 6), match='Center'>
```

```python
text = 'Center is American spelling. Centre is British'

print(re.search(r'Centre', text))
print(re.match(r'Centre', text))
```
```
<re.Match object; span=(29, 35), match='Centre'>
None
```

<br>
### Other Special Characters {#other-special-characters}

The `^` character is used as an anchor to match the start of a string. It is often used in combination with other characters to form a pattern that must occur at the beginning of the string. For example, the pattern `^A` would match any string that starts with an uppercase `A`, while the pattern `^[a-z]` would match any string that starts with a lowercase letter.

The `$` character is used as an anchor to match the end of a string. It is often used in combination with other characters to form a pattern that must occur at the end of the string. For example, the pattern `A$` would match any string that ends with an uppercase `A`, while the pattern `[a-z]$` would match any string that ends with a lowercase letter.

```python
text = 'the 70s music is not as good as the 80s, but my favourite is the 90s'

print(re.findall(r'the\s\d{2}s', text))
print(re.findall(r'^the\s\d{2}s', text))
print(re.findall(r'the\s\d{2}s$', text))
```
```
['the 70s', 'the 80s', 'the 90s']
['the 70s']
['the 90s']
```

<br>
The `|` (pipe) operator is used to specify alternatives. It allows you to match one of multiple possible patterns. For example, the pattern `A|B` would match any string that contains either an uppercase `A` or an uppercase `B`.

You can chain multiple `|` together to match one of several possible patterns.

```python
text = 'The 70s music is not as good as the 80s, but my favourite is the 90s'

print(re.findall(r'the\s\d{2}s', text))
print(re.findall(r'The\s\d{2}s|the\s\d{2}s', text))
```
```
['the 80s', 'the 90s']
['The 70s', 'the 80s', 'the 90s']
```

<br>
Square brackets `[]` are used to define a **character set**, also known as a **character class**. A character class matches any single character that is contained within the square brackets.

```python
text = 'The 70s music is not as good as the 80s, but my favourite is the 90s'

print(re.findall(r'[Tt]he\s\d{2}s', text))
```
```
['The 70s', 'the 80s', 'the 90s']
```

In the example below the regular expression looks for one or more (the `+` symbol) lowercase or uppercase letters (`a-z` or `A-Z`) followed by a whitespace character (`\s`) and then two digits (`\d{2}`) and the letter `s`.

```python
text = 'The 70s music is not as good as the 80s, but my favourite is the 90s'

print(re.findall(r'[a-zA-Z]+\s\d{2}s', text))
```
```
['The 70s', 'the 80s', 'the 90s']
```

<br>
It's also possible to use the caret symbol `^` inside square brackets to negate the character set. Additionally, special characters like the dot, dollar, and caret lose their special meaning when enclosed in square brackets. In the example below the regular expression looks for the `@` symbol followed by one or more characters that are not a number or a comma (using the negated character class `[^0-9,]`).

```python
users = '@admin, @123crazyduck, @999guest'

print(re.findall(r'@[^0-9,]+', users))
```
```
['@admin']
```

<br>
## Greedy vs Non-Greedy Matching {#greedy-non-greedy}

All the quantifiers presented above (`*`, `+`, `?`, `{n,m}`) are **greedy operators** by default. That means they try to match as much text as possible (and so return the longest match possible).  

**Non-greedy (lazy)** quantifiers try to match as few characters as possible, therefore returning the shortest possible match. When a quantifier is followed by a `?`, it becomes non-greedy or lazy.

```python
example = '12345example'

print(re.match(r'\d+', example)) # greedy
print(re.match(r'\d+?', example)) # non-greedy
```
```
<re.Match object; span=(0, 5), match='12345'>
<re.Match object; span=(0, 1), match='1'>
```

In the first example the quantifier matches the first digit and then goes on and searches for more digits. It keeps going and matches more and more digits until no other digit can be matched. Non-greedy quantifier looks for a digit and after it finds the first one it stops, since one digit is as few as we need. 

Non-greedy quantifiers come in handy for removing HTML tags.

```python
html = "Example of an <b>HTML</b> text. Let's remove <i>all tags</i>!"

print(re.sub(r'<.+?>', "", html))
```
```
Example of an HTML text. Let's remove all tags!
```

<br>
## Capturing Groups {#capturing-groups}

Capturing groups is a way to group together parts of a pattern, and then to refer to those groups later in the expression or in the replacement string.

A capturing group is created by placing the pattern inside parentheses `()`. For example, the pattern `(abc)` would match the text `abc` and create a capturing group containing the matched text.

Capturing groups can be referred to by their numbers, with the leftmost group being group 1, the next group being group 2, and so on.

In the example below, the regex looks for information about the name of the pet owner, number of pets and the type of pet.

```python
text = 'All my friends have pets. Anna has 2 dogs. Mark has 2 cats. And George has 101 fish!'

print(re.findall(r'[A-Za-z]+\shas\s\d+\s[a-z]+', text))
```
```
['Anna has 2 dogs', 'Mark has 2 cats', 'George has 101 fish']
```

Not bad. However, we can use groups to get a result which can be easily transformed to pandas DataFrame or other _analysis-friendly_ structure. 

```python
text = 'All my friends have pets. Anna has 2 dogs. Mark has 2 cats. And George has 101 fish!'

print(re.findall(r'([A-Za-z]+)\shas\s(\d+)\s([a-z]+)', text))

# Group 1: ([A-Za-z]+)
# Group 2: (\d+)
# Group 3: ([a-z]+)

# Group 0: ([A-Za-z]+)\shas\s(\d+)\s([a-z]+)
```
```
[('Anna', '2', 'dogs'), ('Mark', '2', 'cats'), ('George', '101', 'fish')]
```

The result is a list of tuples which can easily be converted to other data structures if needed.

This is not the only advantage of groups. The quantifiers normally apply to one character on their immediate left. By capturing a group we can apply a quantifier to a bigger part of regex. 

It is important to note the difference between **capturing a repeated group** and **repeating a capturing group**.

```python
numbers = 'First number is 909. Second number is 1255.'

print(re.findall(r'(\d+)', numbers))
print(re.findall(r'(\d)+', numbers))
```
```
['909', '1255']
['9', '5']
```

The second result might be somewhat surprising and not intuitive. More details about what happens there and why can be found here: [Understanding the (\D\d)+ Regex pattern in Python](https://stackoverflow.com/questions/49079552/understanding-the-d-d-regex-pattern-in-python)

<br>
## Alternation & Non-Capturing Groups {#alternation-non-capturing-groups}

Alternation is a way to match one of several different patterns. It is represented by the vertical bar (pipe) `|` symbol, also known as the OR operator.

Round brackets can be used to group alternative patterns. Suppose you have a list of URLs as below and want to find only those that end with `.org` or `.com`:

```python
urls = 'https://www.wikipedia.org/, https://www.google.com/, ' \
       'https://www.regular-expressions.info/, https://klaudiawolinska.github.io/'

print(re.findall(r'www\.([a-zA-Z0-9]+)\.(org|com)', urls))
```
```
[('wikipedia', 'org'), ('google', 'com')]
```

The last round brackets `(org|com)` were used here to group URLs endings. However, you can use the grouping `()` with the question mark and colon `?:` to match the group, but not capture it. This is called a non-capturing group.

```python
urls = 'https://www.wikipedia.org/, https://www.google.com/, ' \
       'https://www.regular-expressions.info/, https://klaudiawolinska.github.io/'

print(re.findall(r'www\.([a-zA-Z0-9]+)\.(?:org|com)', urls))
```
```
['wikipedia', 'google']
```

<br>
## Named Groups {#named-groups}

Named groups can be created using the syntax `(?P<groupname>pattern)`. Thanks to this, you can reference the groups not only by their numbers but also their names. 

```python
urls = 'https://www.wikipedia.org/, https://www.google.com/, ' \
       'https://www.regular-expressions.info/, https://klaudiawolinska.github.io/'

result = re.search(r'www\.([a-zA-Z0-9]+)\.(?P<top_level_domain>org|com)', urls)

print(result.group(0))
print(result.group(1))
print(result.group(2))
print(result.group('top_level_domain'))
```
```
www.wikipedia.org
wikipedia
org
org
```

<br>
## Backreferences {#backreferences}

Backreferencing in regular expressions allows you to refer back to a previously captured group in the same regular expression. This is done by including a backslash `\` followed by the group number or group name in the pattern.

Backreferencing can be useful for validation, such as checking if a password and its confirmation match, or for more advanced text processing tasks, such as finding duplicate words in a sentence.

```python
text = 'I am so so happy writing this regex paper. ' \
       'I am serious, it makes me very very very happy.'

print(re.findall(r'(\w+)\s\1', text))
```
```
['so', 'very']
```

The regular expression above is a pattern that matches one or more word characters `(\w+)` followed by a whitespace `\s` and then the same exact word as the one that was matched by the first group `\1`.

It's important to note that this pattern will not match the case where a word is repeated more than twice in a row, it just finds the first occurrence of a duplicate word.

This example can be extended to remove duplicated words from the text.

```python
text = 'I am so so happy writing this regex paper. ' \
       'I am serious, it makes me very very very happy.'

print(re.sub(r'(\w+)(?:\s\1)+', r'\1', text))
```
```
I am so happy writing this regex paper. I am serious, it makes me very happy.
```

Note how the plus sign `+` after the non-capturing group indicates that the group should be matched one or more times. In other words it matches one or more consecutive occurrences of the same word.

This example can also be rewritten using named groups. To replace a named capturing group, a special syntax is needed in the replacement field `\g<groupname>`.

```python
text = 'I am so so happy writing this regex paper. ' \
       'I am serious, it makes me very very very happy.'

print(re.sub(r'(?P<repeated>\w+)\s(?P=repeated)', r'\g<repeated>', text))
print(re.sub(r'(?P<repeated>\w+)(?:\s(?P=repeated))+', r'\g<repeated>', text))
```
```
I am so happy writing this regex paper. I am serious, it makes me very very happy.
I am so happy writing this regex paper. I am serious, it makes me very happy.
```

<br>
## Looking Around {#looking-around}

Regex lookaround groups are a special type of non-capturing groups that allow to assert a certain condition before or after a certain pattern without including the matched characters in the final match. They are composed of two types:

* Positive Lookahead `?=pattern` is used to match a pattern only if it's followed by another pattern. It matches the text but does not include it in the final match.

* Negative Lookahead `?!pattern` is used to match a pattern only if it's not followed by another pattern.

* Positive Lookbehind `?<=pattern` is used to match a pattern only if it's preceded by another pattern. It matches the text but does not include it in the final match.

* Negative Lookbehind `?<!pattern` is used to match a pattern only if it's not preceded by another pattern.

<br>
Example 1: This regular expression matches any word characters (alphanumeric and underscore) that occur one or more times, followed by `.txt`, only if it is immediately followed by the string `: success`.

```python
files = 'file1.txt: success, file2.txt: fail, file1.csv: success, file2.xlsx: success'

# Example 1: positive look-ahead
# Find all .txt files that were successfully processed.

print(re.findall(r'\w+\.txt(?=:\ssuccess)', files))
```
```
['file1.txt']
```

<br>
Example 2: This regular expression matches any word characters (alphanumeric and underscore) that occur one or more times, followed by `.txt`, only if it is not immediately followed by the string `: success`.

```python
files = 'file1.txt: success, file2.txt: fail, file1.csv: success, file2.xlsx: success'

# Example 2: negative look-ahead
# Find all .txt files that were not successfully processed.

print(re.findall(r'\w+\.txt(?!:\ssuccess)', files))
```
```
['file2.txt']
```

<br>
Example 3: This regular expression matches any word characters (alphanumeric and underscore) that occur one or more times, only if it is immediately preceded by the string `.txt`, followed by a colon and a space, and then captures one or more word characters immediately following the space into a separate group.

```python
files = 'file1.txt: success, file2.txt: fail, file1.csv: success, file2.xlsx: success'

# Example 3: positive look-behind
# Find all processing statuses of .txt files.

print(re.findall(r'\w+(?<=\.txt):\s(\w+)', files))
```
```
['success', 'fail']
```

<br>
Example 4: This regular expression matches any word characters (alphanumeric and underscore) that occur one or more times, only if it is not immediately preceded by the string `.txt`, followed by a colon and a space, and then captures one or more word characters immediately following the space into a separate group.

```python
files = 'file1.txt: success, file2.txt: fail, file1.csv: success, file2.xlsx: success'

# Example 4: negative look-behind
# Find all processing statuses of files that are not .txt.

print(re.findall(r'\w+(?<!\.txt):\s(\w+)', files))
```
```
['success', 'success']
```

<br>
Congratulations on reaching the end of this post! Regex can be extremely difficult but they're also a very powerful tool.
