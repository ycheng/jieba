jieba
========
“結巴”中文分詞：做最好的 Python 中文分詞組件

"Jieba" (Chinese for "to stutter") Chinese text segmentation: built to be the best Python Chinese word segmentation module.

- _Scroll down for English documentation._


特點
========
* 支持三種分詞模式：
    * 精確模式，試圖將句子最精確地切開，適合文本分析；
    * 全模式，把句子中所有的可以成詞的詞語都掃瞄出來, 速度非常快，但是不能解決歧義；
    * 搜索引擎模式，在精確模式的基礎上，對長詞再次切分，提高召回率，適合用於搜索引擎分詞。

* 支持繁體分詞
* 支持自定義詞典
* MIT 授權協議

在綫演示
=========
http://jiebademo.ap01.aws.af.cm/

(Powered by Appfog)

網站代碼：https://github.com/fxsjy/jiebademo


安裝說明
=======

代碼對 Python 2/3 均兼容

* 全自動安裝：`easy_install jieba` 或者 `pip install jieba` / `pip3 install jieba`
* 半自動安裝：先下載 http://pypi.python.org/pypi/jieba/ ，解壓後運行 `python setup.py install`
* 手動安裝：將 jieba 目錄放置於當前目錄或者 site-packages 目錄
* 通過 `import jieba` 來引用

算法
========
* 基于首碼詞典實現高效的詞圖掃瞄，生成句子中漢字所有可能成詞情況所構成的有向無環圖 (DAG)
* 採用了動態規劃查找最大概率路徑, 找出基于詞頻的最大切分組合
* 對於未登錄詞，採用了基于漢字成詞能力的 HMM 模型，使用了 Viterbi 算法

主要功能
=======
1. 分詞
--------
* `jieba.cut` 方法接受三個輸入參數: 需要分詞的字元串；cut_all 參數用來控制是否採用全模式；HMM 參數用來控制是否使用 HMM 模型
* `jieba.cut_for_search` 方法接受兩個參數：需要分詞的字元串；是否使用 HMM 模型。該方法適合用於搜索引擎構建倒排索引的分詞，粒度比較細
* 待分詞的字元串可以是 unicode 或 UTF-8 字元串、GBK 字元串。注意：不建議直接輸入 GBK 字元串，可能無法預料地錯誤解碼成 UTF-8
* `jieba.cut` 以及 `jieba.cut_for_search` 返回的結構都是一個可迭代的 generator，可以使用 for 循環來獲得分詞後得到的每一個詞語(unicode)，或者用
* `jieba.lcut` 以及 `jieba.lcut_for_search` 直接返回 list
* `jieba.Tokenizer(dictionary=DEFAULT_DICT)` 新建自定義分詞器，可用於同時使用不同詞典。`jieba.dt` 為預設分詞器，所有全局分詞相關函數都是該分詞器的映射。

代碼示例

```python
# encoding=utf-8
import jieba

seg_list = jieba.cut("我來到北京清華大學", cut_all=True)
print("Full Mode: " + "/ ".join(seg_list))  # 全模式

seg_list = jieba.cut("我來到北京清華大學", cut_all=False)
print("Default Mode: " + "/ ".join(seg_list))  # 精確模式

seg_list = jieba.cut("他來到了網易杭研大廈")  # 預設是精確模式
print(", ".join(seg_list))

seg_list = jieba.cut_for_search("小明碩士畢業于中國科學院計算所，後在日本京都大學深造")  # 搜索引擎模式
print(", ".join(seg_list))
```

輸出:

    【全模式】: 我/ 來到/ 北京/ 清華/ 清華大學/ 華大/ 大學

    【精確模式】: 我/ 來到/ 北京/ 清華大學

    【新詞識別】：他, 來到, 了, 網易, 杭研, 大廈    (此處，“杭研”並沒有在詞典中，但是也被Viterbi算法識別出來了)

    【搜索引擎模式】： 小明, 碩士, 畢業, 于, 中國, 科學, 學院, 科學院, 中國科學院, 計算, 計算所, 後, 在, 日本, 京都, 大學, 日本京都大學, 深造

2. 添加自定義詞典
----------------

### 載入詞典

* 開發者可以指定自己自定義的詞典，以便包含 jieba 詞庫裡沒有的詞。雖然 jieba 有新詞識別能力，但是自行添加新詞可以保證更高的正確率
* 用法： jieba.load_userdict(file_name) # file_name 為檔案類對象或自定義詞典的路徑
* 詞典格式和 `dict.txt` 一樣，一個詞占一行；每一行分三部分：詞語、詞頻（可省略）、詞性（可省略），用空格隔開，順序不可顛倒。`file_name` 若為路徑或二進制方式打開的檔案，則檔案必須為 UTF-8 編碼。
* 詞頻省略時使用自動計算的能保證分出該詞的詞頻。

**例如：**

```
創新辦 3 i
雲計算 5
凱特琳 nz
台中
```

* 更改分詞器（預設為 `jieba.dt`）的 `tmp_dir` 和 `cache_file` 屬性，可分別指定緩存檔案所在的檔案夾及其檔案名，用於受限的檔案系統。

* 範例：

    * 自定義詞典：https://github.com/fxsjy/jieba/blob/master/test/userdict.txt

    * 用法示例：https://github.com/fxsjy/jieba/blob/master/test/test_userdict.py


        * 之前： 李小福 / 是 / 創新 / 辦 / 主任 / 也 / 是 / 雲 / 計算 / 方面 / 的 / 專家 /

        * 加載自定義詞庫後：　李小福 / 是 / 創新辦 / 主任 / 也 / 是 / 雲計算 / 方面 / 的 / 專家 /

### 調整詞典

* 使用 `add_word(word, freq=None, tag=None)` 和 `del_word(word)` 可在程序中動態修改詞典。
* 使用 `suggest_freq(segment, tune=True)` 可調節單個詞語的詞頻，使其能（或不能）被分出來。

* 注意：自動計算的詞頻在使用 HMM 新詞發現功能時可能無效。

代碼示例：

```pycon
>>> print('/'.join(jieba.cut('如果放到post中將出錯。', HMM=False)))
如果/放到/post/中將/出錯/。
>>> jieba.suggest_freq(('中', '將'), True)
494
>>> print('/'.join(jieba.cut('如果放到post中將出錯。', HMM=False)))
如果/放到/post/中/將/出錯/。
>>> print('/'.join(jieba.cut('「台中」正確應該不會被切開', HMM=False)))
「/台/中/」/正確/應該/不會/被/切開
>>> jieba.suggest_freq('台中', True)
69
>>> print('/'.join(jieba.cut('「台中」正確應該不會被切開', HMM=False)))
「/台中/」/正確/應該/不會/被/切開
```

* "通過用戶自定義詞典來增強歧義糾錯能力" --- https://github.com/fxsjy/jieba/issues/14

3. 關鍵詞提取
-------------
### 基于 TF-IDF 算法的關鍵詞抽取

`import jieba.analyse`

* jieba.analyse.extract_tags(sentence, topK=20, withWeight=False, allowPOS=())
  * sentence 為待提取的文本
  * topK 為返回幾個 TF/IDF 權重最大的關鍵詞，預設值為 20
  * withWeight 為是否一併返回關鍵詞權重值，預設值為 False
  * allowPOS 僅包括指定詞性的詞，預設值為空，即不篩選
* jieba.analyse.TFIDF(idf_path=None) 新建 TFIDF 實例，idf_path 為 IDF 頻率檔案

代碼示例 （關鍵詞提取）

https://github.com/fxsjy/jieba/blob/master/test/extract_tags.py

關鍵詞提取所使用逆向檔案頻率（IDF）文本語料庫可以切換成自定義語料庫的路徑

* 用法： jieba.analyse.set_idf_path(file_name) # file_name為自定義語料庫的路徑
* 自定義語料庫示例：https://github.com/fxsjy/jieba/blob/master/extra_dict/idf.txt.big
* 用法示例：https://github.com/fxsjy/jieba/blob/master/test/extract_tags_idfpath.py

關鍵詞提取所使用停止詞（Stop Words）文本語料庫可以切換成自定義語料庫的路徑

* 用法： jieba.analyse.set_stop_words(file_name) # file_name為自定義語料庫的路徑
* 自定義語料庫示例：https://github.com/fxsjy/jieba/blob/master/extra_dict/stop_words.txt
* 用法示例：https://github.com/fxsjy/jieba/blob/master/test/extract_tags_stop_words.py

關鍵詞一併返回關鍵詞權重值示例

* 用法示例：https://github.com/fxsjy/jieba/blob/master/test/extract_tags_with_weight.py

### 基于 TextRank 算法的關鍵詞抽取

* jieba.analyse.textrank(sentence, topK=20, withWeight=False, allowPOS=('ns', 'n', 'vn', 'v')) 直接使用，介面相同，注意預設過濾詞性。
* jieba.analyse.TextRank() 新建自定義 TextRank 實例

算法論文： [TextRank: Bringing Order into Texts](http://web.eecs.umich.edu/~mihalcea/papers/mihalcea.emnlp04.pdf)

#### 基本思想:

1. 將待抽取關鍵詞的文本進行分詞
2. 以固定窗口大小(預設為5，通過span屬性調整)，詞之間的共現關係，構建圖
3. 計算圖中節點的PageRank，注意是無向帶權圖

#### 使用示例:

見 [test/demo.py](https://github.com/fxsjy/jieba/blob/master/test/demo.py)

4. 詞性標註
-----------
* `jieba.posseg.POSTokenizer(tokenizer=None)` 新建自定義分詞器，`tokenizer` 參數可指定內部使用的 `jieba.Tokenizer` 分詞器。`jieba.posseg.dt` 為預設詞性標註分詞器。
* 標註句子分詞後每個詞的詞性，採用和 ictclas 兼容的標記法。
* 用法示例

```pycon
>>> import jieba.posseg as pseg
>>> words = pseg.cut("我愛北京天安門")
>>> for word, flag in words:
...    print('%s %s' % (word, flag))
...
我 r
愛 v
北京 ns
天安門 ns
```

5. 並行分詞
-----------
* 原理：將目標文本按行分隔後，把各行文本分配到多個 Python 進程並行分詞，然後歸併結果，從而獲得分詞速度的可觀提升
* 基于 python 自帶的 multiprocessing 模組，目前暫不支持 Windows
* 用法：
    * `jieba.enable_parallel(4)` # 開啟並行分詞模式，參數為並行進程數
    * `jieba.disable_parallel()` # 關閉並行分詞模式

* 例子：https://github.com/fxsjy/jieba/blob/master/test/parallel/test_file.py

* 實驗結果：在 4 核 3.4GHz Linux 機器上，對金庸全集進行精確分詞，獲得了 1MB/s 的速度，是單進程版的 3.3 倍。

* **注意**：並行分詞僅支持預設分詞器 `jieba.dt` 和 `jieba.posseg.dt`。

6. Tokenize：返回詞語在原文的起止位置
----------------------------------
* 注意，輸入參數隻接受 unicode
* 預設模式

```python
result = jieba.tokenize(u'永和服裝飾品有限公司')
for tk in result:
    print("word %s\t\t start: %d \t\t end:%d" % (tk[0],tk[1],tk[2]))
```

```
word 永和                start: 0                end:2
word 服裝                start: 2                end:4
word 飾品                start: 4                end:6
word 有限公司            start: 6                end:10

```

* 搜索模式

```python
result = jieba.tokenize(u'永和服裝飾品有限公司', mode='search')
for tk in result:
    print("word %s\t\t start: %d \t\t end:%d" % (tk[0],tk[1],tk[2]))
```

```
word 永和                start: 0                end:2
word 服裝                start: 2                end:4
word 飾品                start: 4                end:6
word 有限                start: 6                end:8
word 公司                start: 8                end:10
word 有限公司            start: 6                end:10
```


7. ChineseAnalyzer for Whoosh 搜索引擎
--------------------------------------------
* 引用： `from jieba.analyse import ChineseAnalyzer`
* 用法示例：https://github.com/fxsjy/jieba/blob/master/test/test_whoosh.py

8. 命令行分詞
-------------------

使用示例：`python -m jieba news.txt > cut_result.txt`

命令行選項（翻譯）：

    使用: python -m jieba [options] filename

    結巴命令行界面。

    固定參數:
      filename              輸入檔案

    可選參數:
      -h, --help            顯示此幫助信息並退出
      -d [DELIM], --delimiter [DELIM]
                            使用 DELIM 分隔詞語，而不是用預設的' / '。
                            若不指定 DELIM，則使用一個空格分隔。
      -p [DELIM], --pos [DELIM]
                            啟用詞性標註；如果指定 DELIM，詞語和詞性之間
                            用它分隔，否則用 _ 分隔
      -D DICT, --dict DICT  使用 DICT 代替預設詞典
      -u USER_DICT, --user-dict USER_DICT
                            使用 USER_DICT 作為附加詞典，與預設詞典或自定義詞典配合使用
      -a, --cut-all         全模式分詞（不支持詞性標註）
      -n, --no-hmm          不使用隱含馬爾可夫模型
      -q, --quiet           不輸出載入信息到 STDERR
      -V, --version         顯示版本信息並退出

    如果沒有指定檔案名，則使用標準輸入。

`--help` 選項輸出：

    $> python -m jieba --help
    Jieba command line interface.

    positional arguments:
      filename              input file

    optional arguments:
      -h, --help            show this help message and exit
      -d [DELIM], --delimiter [DELIM]
                            use DELIM instead of ' / ' for word delimiter; or a
                            space if it is used without DELIM
      -p [DELIM], --pos [DELIM]
                            enable POS tagging; if DELIM is specified, use DELIM
                            instead of '_' for POS delimiter
      -D DICT, --dict DICT  use DICT as dictionary
      -u USER_DICT, --user-dict USER_DICT
                            use USER_DICT together with the default dictionary or
                            DICT (if specified)
      -a, --cut-all         full pattern cutting (ignored with POS tagging)
      -n, --no-hmm          don't use the Hidden Markov Model
      -q, --quiet           don't print loading messages to stderr
      -V, --version         show program's version number and exit

    If no filename specified, use STDIN instead.

延遲加載機制
------------

jieba 採用延遲加載，`import jieba` 和 `jieba.Tokenizer()` 不會立即觸發詞典的加載，一旦有必要才開始加載詞典構建首碼字典。如果你想手工初始 jieba，也可以手動初始化。

    import jieba
    jieba.initialize()  # 手動初始化（可選）


在 0.28 之前的版本是不能指定主詞典的路徑的，有了延遲加載機制後，你可以改變主詞典的路徑:

    jieba.set_dictionary('data/dict.txt.big')

例子： https://github.com/fxsjy/jieba/blob/master/test/test_change_dictpath.py

其他詞典
========
1. 占用內存較小的詞典檔案
https://github.com/fxsjy/jieba/raw/master/extra_dict/dict.txt.small

2. 支持繁體分詞更好的詞典檔案
https://github.com/fxsjy/jieba/raw/master/extra_dict/dict.txt.big

下載你所需要的詞典，然後覆蓋 jieba/dict.txt 即可；或者用 `jieba.set_dictionary('data/dict.txt.big')`

其他語言實現
==========

結巴分詞 Java 版本
----------------
作者：piaolingxue
地址：https://github.com/huaban/jieba-analysis

結巴分詞 C++ 版本
----------------
作者：yanyiwu
地址：https://github.com/yanyiwu/cppjieba

結巴分詞 Node.js 版本
----------------
作者：yanyiwu
地址：https://github.com/yanyiwu/nodejieba

結巴分詞 Erlang 版本
----------------
作者：falood
地址：https://github.com/falood/exjieba

結巴分詞 R 版本
----------------
作者：qinwf
地址：https://github.com/qinwf/jiebaR

結巴分詞 iOS 版本
----------------
作者：yanyiwu
地址：https://github.com/yanyiwu/iosjieba

結巴分詞 PHP 版本
----------------
作者：fukuball
地址：https://github.com/fukuball/jieba-php

結巴分詞 .NET(C#) 版本
----------------
作者：anderscui
地址：https://github.com/anderscui/jieba.NET/

結巴分詞 Go 版本
----------------

+ 作者: wangbin 地址: https://github.com/wangbin/jiebago
+ 作者: yanyiwu 地址: https://github.com/yanyiwu/gojieba

系統整合
========
1. Solr: https://github.com/sing1ee/jieba-solr

分詞速度
=========
* 1.5 MB / Second in Full Mode
* 400 KB / Second in Default Mode
* 測試環境: Intel(R) Core(TM) i7-2600 CPU @ 3.4GHz；《圍城》.txt

常見問題
=========

## 1. 模型的數據是如何生成的？

詳見： https://github.com/fxsjy/jieba/issues/7

## 2. “台中”總是被切成“台 中”？（以及類似情況）

P(台中) ＜ P(台)×P(中)，“台中”詞頻不夠導致其成詞概率較低

解決方法：強制調高詞頻

`jieba.add_word('台中')` 或者 `jieba.suggest_freq('台中', True)`

## 3. “今天天氣 不錯”應該被切成“今天 天氣 不錯”？（以及類似情況）

解決方法：強制調低詞頻

`jieba.suggest_freq(('今天', '天氣'), True)`

或者直接刪除該詞 `jieba.del_word('今天天氣')`

## 4. 切出了詞典中沒有的詞語，效果不理想？

解決方法：關閉新詞發現

`jieba.cut('豐田太省了', HMM=False)`
`jieba.cut('我們中出了一個叛徒', HMM=False)`

**更多問題請點擊**：https://github.com/fxsjy/jieba/issues?sort=updated&state=closed

修訂歷史
==========
https://github.com/fxsjy/jieba/blob/master/Changelog

--------------------

jieba
========
"Jieba" (Chinese for "to stutter") Chinese text segmentation: built to be the best Python Chinese word segmentation module.

Features
========
* Support three types of segmentation mode:

1. Accurate Mode attempts to cut the sentence into the most accurate segmentations, which is suitable for text analysis.
2. Full Mode gets all the possible words from the sentence. Fast but not accurate.
3. Search Engine Mode, based on the Accurate Mode, attempts to cut long words into several short words, which can raise the recall rate. Suitable for search engines.

* Supports Traditional Chinese
* Supports customized dictionaries
* MIT License


Online demo
=========
http://jiebademo.ap01.aws.af.cm/

(Powered by Appfog)

Usage
========
* Fully automatic installation: `easy_install jieba` or `pip install jieba`
* Semi-automatic installation: Download http://pypi.python.org/pypi/jieba/ , run `python setup.py install` after extracting.
* Manual installation: place the `jieba` directory in the current directory or python `site-packages` directory.
* `import jieba`.

Algorithm
========
* Based on a prefix dictionary structure to achieve efficient word graph scanning. Build a directed acyclic graph (DAG) for all possible word combinations.
* Use dynamic programming to find the most probable combination based on the word frequency.
* For unknown words, a HMM-based model is used with the Viterbi algorithm.

Main Functions
==============

1. Cut
--------
* The `jieba.cut` function accepts three input parameters: the first parameter is the string to be cut; the second parameter is `cut_all`, controlling the cut mode; the third parameter is to control whether to use the Hidden Markov Model.
* `jieba.cut_for_search` accepts two parameter: the string to be cut; whether to use the Hidden Markov Model. This will cut the sentence into short words suitable for search engines.
* The input string can be an unicode/str object, or a str/bytes object which is encoded in UTF-8 or GBK. Note that using GBK encoding is not recommended because it may be unexpectly decoded as UTF-8.
* `jieba.cut` and `jieba.cut_for_search` returns an generator, from which you can use a `for` loop to get the segmentation result (in unicode).
* `jieba.lcut` and `jieba.lcut_for_search` returns a list.
* `jieba.Tokenizer(dictionary=DEFAULT_DICT)` creates a new customized Tokenizer, which enables you to use different dictionaries at the same time. `jieba.dt` is the default Tokenizer, to which almost all global functions are mapped.


**Code example: segmentation**

```python
#encoding=utf-8
import jieba

seg_list = jieba.cut("我來到北京清華大學", cut_all=True)
print("Full Mode: " + "/ ".join(seg_list))  # 全模式

seg_list = jieba.cut("我來到北京清華大學", cut_all=False)
print("Default Mode: " + "/ ".join(seg_list))  # 預設模式

seg_list = jieba.cut("他來到了網易杭研大廈")
print(", ".join(seg_list))

seg_list = jieba.cut_for_search("小明碩士畢業于中國科學院計算所，後在日本京都大學深造")  # 搜索引擎模式
print(", ".join(seg_list))
```

Output:

    [Full Mode]: 我/ 來到/ 北京/ 清華/ 清華大學/ 華大/ 大學

    [Accurate Mode]: 我/ 來到/ 北京/ 清華大學

    [Unknown Words Recognize] 他, 來到, 了, 網易, 杭研, 大廈    (In this case, "杭研" is not in the dictionary, but is identified by the Viterbi algorithm)

    [Search Engine Mode]： 小明, 碩士, 畢業, 于, 中國, 科學, 學院, 科學院, 中國科學院, 計算, 計算所, 後, 在, 日本, 京都, 大學, 日本京都大學, 深造


2. Add a custom dictionary
----------------------------

### Load dictionary

* Developers can specify their own custom dictionary to be included in the jieba default dictionary. Jieba is able to identify new words, but you can add your own new words can ensure a higher accuracy.
* Usage： `jieba.load_userdict(file_name)` # file_name is a file-like object or the path of the custom dictionary
* The dictionary format is the same as that of `dict.txt`: one word per line; each line is divided into three parts separated by a space: word, word frequency, POS tag. If `file_name` is a path or a file opened in binary mode, the dictionary must be UTF-8 encoded.
* The word frequency and POS tag can be omitted respectively. The word frequency will be filled with a suitable value if omitted.

**For example:**

```
創新辦 3 i
雲計算 5
凱特琳 nz
台中
```


* Change a Tokenizer's `tmp_dir` and `cache_file` to specify the path of the cache file, for using on a restricted file system.

* Example:

        雲計算 5
        李小福 2
        創新辦 3

        [Before]： 李小福 / 是 / 創新 / 辦 / 主任 / 也 / 是 / 雲 / 計算 / 方面 / 的 / 專家 /

        [After]：　李小福 / 是 / 創新辦 / 主任 / 也 / 是 / 雲計算 / 方面 / 的 / 專家 /


### Modify dictionary

* Use `add_word(word, freq=None, tag=None)` and `del_word(word)` to modify the dictionary dynamically in programs.
* Use `suggest_freq(segment, tune=True)` to adjust the frequency of a single word so that it can (or cannot) be segmented.

* Note that HMM may affect the final result.

Example:

```pycon
>>> print('/'.join(jieba.cut('如果放到post中將出錯。', HMM=False)))
如果/放到/post/中將/出錯/。
>>> jieba.suggest_freq(('中', '將'), True)
494
>>> print('/'.join(jieba.cut('如果放到post中將出錯。', HMM=False)))
如果/放到/post/中/將/出錯/。
>>> print('/'.join(jieba.cut('「台中」正確應該不會被切開', HMM=False)))
「/台/中/」/正確/應該/不會/被/切開
>>> jieba.suggest_freq('台中', True)
69
>>> print('/'.join(jieba.cut('「台中」正確應該不會被切開', HMM=False)))
「/台中/」/正確/應該/不會/被/切開
```

3. Keyword Extraction
-----------------------
`import jieba.analyse`

* `jieba.analyse.extract_tags(sentence, topK=20, withWeight=False, allowPOS=())`
  * `sentence`: the text to be extracted
  * `topK`: return how many keywords with the highest TF/IDF weights. The default value is 20
  * `withWeight`: whether return TF/IDF weights with the keywords. The default value is False
  * `allowPOS`: filter words with which POSs are included. Empty for no filtering.
* `jieba.analyse.TFIDF(idf_path=None)` creates a new TFIDF instance, `idf_path` specifies IDF file path.

Example (keyword extraction)

https://github.com/fxsjy/jieba/blob/master/test/extract_tags.py

Developers can specify their own custom IDF corpus in jieba keyword extraction

* Usage： `jieba.analyse.set_idf_path(file_name) # file_name is the path for the custom corpus`
* Custom Corpus Sample：https://github.com/fxsjy/jieba/blob/master/extra_dict/idf.txt.big
* Sample Code：https://github.com/fxsjy/jieba/blob/master/test/extract_tags_idfpath.py

Developers can specify their own custom stop words corpus in jieba keyword extraction

* Usage： `jieba.analyse.set_stop_words(file_name) # file_name is the path for the custom corpus`
* Custom Corpus Sample：https://github.com/fxsjy/jieba/blob/master/extra_dict/stop_words.txt
* Sample Code：https://github.com/fxsjy/jieba/blob/master/test/extract_tags_stop_words.py

There's also a [TextRank](http://web.eecs.umich.edu/~mihalcea/papers/mihalcea.emnlp04.pdf) implementation available.

Use: `jieba.analyse.textrank(sentence, topK=20, withWeight=False, allowPOS=('ns', 'n', 'vn', 'v'))`

Note that it filters POS by default.

`jieba.analyse.TextRank()` creates a new TextRank instance.

4. Part of Speech Tagging
-------------------------
* `jieba.posseg.POSTokenizer(tokenizer=None)` creates a new customized Tokenizer. `tokenizer` specifies the jieba.Tokenizer to internally use. `jieba.posseg.dt` is the default POSTokenizer.
* Tags the POS of each word after segmentation, using labels compatible with ictclas.
* Example:

```pycon
>>> import jieba.posseg as pseg
>>> words = pseg.cut("我愛北京天安門")
>>> for w in words:
...    print('%s %s' % (w.word, w.flag))
...
我 r
愛 v
北京 ns
天安門 ns
```

5. Parallel Processing
----------------------
* Principle: Split target text by line, assign the lines into multiple Python processes, and then merge the results, which is considerably faster.
* Based on the multiprocessing module of Python.
* Usage:
    * `jieba.enable_parallel(4)` # Enable parallel processing. The parameter is the number of processes.
    * `jieba.disable_parallel()` # Disable parallel processing.

* Example:
    https://github.com/fxsjy/jieba/blob/master/test/parallel/test_file.py

* Result: On a four-core 3.4GHz Linux machine, do accurate word segmentation on Complete Works of Jin Yong, and the speed reaches 1MB/s, which is 3.3 times faster than the single-process version.

* **Note** that parallel processing supports only default tokenizers, `jieba.dt` and `jieba.posseg.dt`.

6. Tokenize: return words with position
----------------------------------------
* The input must be unicode
* Default mode

```python
result = jieba.tokenize(u'永和服裝飾品有限公司')
for tk in result:
    print("word %s\t\t start: %d \t\t end:%d" % (tk[0],tk[1],tk[2]))
```

```
word 永和                start: 0                end:2
word 服裝                start: 2                end:4
word 飾品                start: 4                end:6
word 有限公司            start: 6                end:10

```

* Search mode

```python
result = jieba.tokenize(u'永和服裝飾品有限公司',mode='search')
for tk in result:
    print("word %s\t\t start: %d \t\t end:%d" % (tk[0],tk[1],tk[2]))
```

```
word 永和                start: 0                end:2
word 服裝                start: 2                end:4
word 飾品                start: 4                end:6
word 有限                start: 6                end:8
word 公司                start: 8                end:10
word 有限公司            start: 6                end:10
```


7. ChineseAnalyzer for Whoosh
-------------------------------
* `from jieba.analyse import ChineseAnalyzer`
* Example: https://github.com/fxsjy/jieba/blob/master/test/test_whoosh.py

8. Command Line Interface
--------------------------------

    $> python -m jieba --help
    Jieba command line interface.

    positional arguments:
      filename              input file

    optional arguments:
      -h, --help            show this help message and exit
      -d [DELIM], --delimiter [DELIM]
                            use DELIM instead of ' / ' for word delimiter; or a
                            space if it is used without DELIM
      -p [DELIM], --pos [DELIM]
                            enable POS tagging; if DELIM is specified, use DELIM
                            instead of '_' for POS delimiter
      -D DICT, --dict DICT  use DICT as dictionary
      -u USER_DICT, --user-dict USER_DICT
                            use USER_DICT together with the default dictionary or
                            DICT (if specified)
      -a, --cut-all         full pattern cutting (ignored with POS tagging)
      -n, --no-hmm          don't use the Hidden Markov Model
      -q, --quiet           don't print loading messages to stderr
      -V, --version         show program's version number and exit

    If no filename specified, use STDIN instead.

Initialization
---------------
By default, Jieba don't build the prefix dictionary unless it's necessary. This takes 1-3 seconds, after which it is not initialized again. If you want to initialize Jieba manually, you can call:

    import jieba
    jieba.initialize()  # (optional)

You can also specify the dictionary (not supported before version 0.28) :

    jieba.set_dictionary('data/dict.txt.big')


Using Other Dictionaries
===========================

It is possible to use your own dictionary with Jieba, and there are also two dictionaries ready for download:

1. A smaller dictionary for a smaller memory footprint:
https://github.com/fxsjy/jieba/raw/master/extra_dict/dict.txt.small

2. There is also a bigger dictionary that has better support for traditional Chinese (繁體):
https://github.com/fxsjy/jieba/raw/master/extra_dict/dict.txt.big

By default, an in-between dictionary is used, called `dict.txt` and included in the distribution.

In either case, download the file you want, and then call `jieba.set_dictionary('data/dict.txt.big')` or just replace the existing `dict.txt`.

Segmentation speed
=========
* 1.5 MB / Second in Full Mode
* 400 KB / Second in Default Mode
* Test Env: Intel(R) Core(TM) i7-2600 CPU @ 3.4GHz；《圍城》.txt

