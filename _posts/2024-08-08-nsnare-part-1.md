---
title: Nsnare File Integrity Checker
date: 2024-09-09
categories: [Projects, nsnare]
tags: [github, python, linux, cybersecurity]
---

After getting my CompTIA A+ and Network+ certifications, I decided my next projects would be to start studying for the Security+, brush up on my Python, and finally start work on my home lab. For now we will focus on the Python bit.

My first foray into Python security tools is [Nsnare](https://github.com/Neyote/nsnare). It's a very simple file integrity checker. It has two main functions:

1. Create a baseline file to reference for integrity checks.
2. Run an integrity check by comparing the current state of the target directory to the baseline file.

I started by going through [this tutorial by @sebastienwebdev](https://medium.com/@sebastienwebdev/file-integrity-monitor-in-python-a-beginners-guide-fedefc9d9284) but once I made it to the monitoring section of the post, I decided I wanted it to run a check instead of constantly monitor for changes. Below I will go through how I handle those checks. The next code snippets will be taken from the `check_integrity()` function definition.

```python
    target_directory = './files_to_scan'
    baseline_dictionary, current_dictionary = {}, {}
    changed_files, removed_files, added_files = [], [], []
    files = [x for x in os.listdir(target_directory)
             if os.path.isfile(os.path.join(target_directory, x))
             and not x.startswith('.')]
```
{: file="main.py"}

First, we set a target directory for the scanner, and initialize some empty dictionaries and arrays. The two dictionaries will hold file paths and the files' corresponding hashes; one for the baseline information, and one for the current state of the target directory. The arrays will hold information we want this function to return. We'll also define a variable that returns an array of valid file paths in the target directory (ones that don't start with '.'), which will be used to populate the current state dictionary later.

```python
    with open('nsnarebaseline.txt', 'r') as baseline_file:
        for line in baseline_file.readlines():
            file_path = line.split('|')[0]
            file_hash = line.split('|')[1].strip('\n')
            baseline_dictionary[file_path] = file_hash
```
{: file="main.py"}

Here we go through the baseline file and fill a dictionary with the file's path as the key, and the hash as a value.
> **Ex.** `{"./files_to_scan/example.txt": "0354b484489432fe8aebec7d1de7df48902a63a347a830cbb504913b8ce22b24df5fe9a3a9ce702087145b28b168f74e5a4cd8d34528f6ee59d5982af763d688"}`

```python
    for x in files:
        file_path = os.path.join(target_directory, x)
        file_hash = calculate_file_hash(file_path)
        current_dictionary[file_path] = file_hash
```
{: file="main.py"}
Now we go through and get the current state of the target directory. We'll grab the file's path and hash the file, then put that information in a dictionary formatted just like the baseline dictionary.

```python
    removed_files = [x for x in baseline_dictionary.keys()
                     if x not in current_dictionary]
    for current_file, current_file_hash in current_dictionary.items():
        if current_file not in baseline_dictionary:
            added_files.append(current_file)
        elif baseline_dictionary[current_file] != current_file_hash:
            changed_files.append(current_file)
        else:
            continue
```
{: file="main.py"}
Here we finally get to the checks. Using a list comprehension, we'll compare the baseline against the current state to find any files that have been removed. Then in the `for` loop we'll compare the current state against the baseline to find new or changed files.

```python
    return added_files, removed_files, changed_files
```
{: file="main.py"}
Lastly we want the function to return its results. They are referenced later when printing out information for the user.

Like I said, fairly simple. I think this was a fun way of starting to re-familiarize myself with Python, and I plan on writing more tools similar to this. The full project is on my GitHub. :)
