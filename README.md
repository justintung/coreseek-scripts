# coreseek辅助脚本

之前这些脚本都贴在gist上，因为众所周知的原因，目前转移至此处，详细说明见文章[使用搜狗词库制作mmseg自定义词典](http://blog.atime.me/note/mmseg-custom-dict.html)。

使用搜狗词库制作mmseg自定义词典

总结使用搜狗词库制作mmseg词典的方法和步骤。另外，最近一直没写新博客，一方面是因为懒，另一方面是确实没什么可写的。

coreseek的介绍和安装説明可参考这里，不再赘述。以下是接下来需要注意的几点:

下面的脚本需要Python2.7+，如果使用时遇到问题请先查看Python的版本(python -V)。
下面假设libmmseg安装于/usr/local/mmseg3目录
生成的mmseg词典文件必须为UTF-8编码。
提取搜狗字库
目前只支持搜狗词库，搜狗词库可以在这里这里下载。

下载后的词库文件(.scel)放在sougou目录下，然后运行脚本

./extract-sougou-dict.py sougou/*.scel -o sougou-dict.txt -mmseg
生成的sougou-dict.txt可供mmseg库分词使用。

合并已有的词典和自定义词典
假设已有的词典文件为unigram.txt，则运行脚本

./merge-mmseg-dict.py -a unigram.txt -b sougou-dict.txt -o merged.txt
这里以unigram.txt为主词典，意味着如果在合并的过程中出现重复词组，则忽略sougou-dict.txt 中的重复词组。

更新mmseg词典
首先备份下旧的词典文件

cd /usr/local/mmseg3/etc
mv unigram.txt unigram.txt.bak
mv uni.lib uni.lib.bak
cd -
将合并后的词典重命名为unigram.txt，在指定的目录里运行mmseg

mv merged.txt /usr/local/mmseg3/etc/unigram.txt
/usr/local/mmseg3/bin/mmseg -u /usr/local/mmseg3/etc/unigram.txt
最后将生成的unigram.txt.uni重命名为uni.lib

cd /usr/local/mmseg3/etc
mv unigram.txt.uni uni.lib
需要注意一个比较蛋疼的问题，更新后的分词效果对已建立的索引无效1。

检验分词效果
找一个之前无法分词的词组，然后用mmseg进行测试，这里假设词典位于/usr/local/mmseg3/etc目录下

echo "金交所" > whatever.txt
/usr/local/mmseg3/bin/mmseg -d /usr/local/mmseg3/etc whatever.txt
如果词典中没有`金交所'这个词组，结果通常如下

金/x 交/x 所/x

Word Splite took: 0 ms.
如果在自定义的词典中有`金交所'这个词组，则结果如下

金交所/x

Word Splite took: 0 ms.
辅助脚本
以下两个脚本放在Github上。

extract-sougou-dict.py

参考了这篇文章里的代码。

merge-mmseg-dict.py

合并两个mmseg词典，如果两个词典包含相同的词组，那么以主词典为准(-a)。

