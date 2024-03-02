## for循环
按照顺序迭代序列中的子元素。如：
```python
words = ['cat', 'window', 'defenestrate']
for w in words:
    print(w, len(w))
```
在循环过程中修改被迭代的对象是不安全的 ，如果要修改迭代的对象，可以迭代其副本。如：
```python
words = ['cat', 'window', 'defenestrate']
for w in words[:]:
	if (len(w) > 6):
		words.insert(0, w)
```
 - [ ] [[为什么在循环过程中修改被迭代对象是不安全的]] #python/why
