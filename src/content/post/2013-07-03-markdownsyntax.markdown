---

comments: true
date: 2013-07-03T00:00:00Z
title: MarkdownSyntax
url: /2013/07/03/markdownsyntax/
---

1\. 自动编号的问题:

Markdown里的自动编号只适用于块(block)中的自动编号，也就是说，在使用了例如Solarized格式化代码块以后，全局中的自动编号不再起作用。
正常的自动编号, 不会有问题, 前标号分别为1/2/3:
```
1. item 1
2. item 2 
3. item 3
```
加入代码块后的情形:
 item 1/item 2/item 3都将以1. item开头, 前标号变成1/1/1:
```
1. item 1
// code blocks
2. item 2 
// code blocks
3. item 3
```
解决方法：使用Markdown转义字符, 在.前加\\
```
1\. item 1
2\. item 2 
3\. item 3
```

2\. Solarized高亮代码的两种用法:

把代码放在如下块中:
{% raw %}
	
```
	//Your code here!
	
```

	或者

	```
	//Your code here!
	```
{% endraw %}

代码高亮方法

a\. 指定codeblock参数：
{% raw %}
	``` 
	#include <stdio.h>
	int main(void)
	{
		printf("Hello World!\n");
		return 0;
	}
	```
{% endraw %}

结果：
```
#include <stdio.h>
int main(void)
{
	printf("Hello World!\n");
	return 0;
}
```


b\. 指定多行代码参数:

标题"Discover if a number is prime", 链接地址"http://...", 链接名称"Source Article".
{% raw %}
	
```
	class Fixnum
	  def prime?
	    ('1' * self) !~ /^1?$|^(11+?)\1+$/
	  end
	end
	
```
{% endraw %}


```
class Fixnum
  def prime?
    ('1' * self) !~ /^1?$|^(11+?)\1+$/
  end
end

```

3\. Syntax Highlight 在 ArchLinux 上的问题：

因为ArchLinux默认的python版本是3, 导致pygments不能正确解析源文件，编辑：
	vim ~/.rvm/gems/ruby-1.9.3-p448/gems/pygments.rb-0.3.4/lib/pygments/mentos.py
	第一行
	/usr/bin/env python --> /usr/bin/env python2
