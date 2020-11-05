1.文件取差集
===========
uniq-u是指拿只出现一次的

只在a中的
sort a b b | uniq -u
只在a中或者只在b中
sort a b | uniq -u
