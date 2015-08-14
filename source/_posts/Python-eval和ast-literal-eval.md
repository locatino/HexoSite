title: Python eval和ast.literal_eval
date: 2015-07-21 11:41:55
tags:
- Python
categories: Memo
---
```datamap = eval(raw_input('Provide some data here: ')``` means that you actually evaluate the code before you deem it to be unsafe or not. It evaluates the code as soon as the function is called. See also the dangers of ```eval```.

```ast.literal_eval``` raises an exception if the input isn't a valid Python datatype, so the code won't be executed if it's not.

__Use ```ast.literal_eval``` whenever you need ```eval```.__ If you have Python expressions as an input that you want to evaluate, you shouldn't (have them).

---
简而言之一句话，```literal_eval```做了类型检查，如果参数无效的话就会报错，而```eval```不会。所以尽量使用```literal_eval```。
