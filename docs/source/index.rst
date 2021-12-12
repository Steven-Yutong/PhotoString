.. PhotoString documentation master file, created by
   sphinx-quickstart on Sun Dec 12 00:04:20 2021.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to PhotoString's documentation!
=======================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:



==================

Lab2：Use blueprints to architect a web application
==============================================

小组成员信息： 

201932110143 王炫 

201932110145 邬程峰 

201932110146 吴彬宇

201932110147 吴雨桐 

201932110148 谢铭轩

项目Github地址：\ `Github <https://github.com/Steven-Yutong/PhotoString.git>`__

项目Read the Docs地址：\ `Read the Docs <https://photostring-yutong.readthedocs.io/en/latest/>`__

Abstract
--------

借助工具研究EnglishPal现有的模块(或类)之间的依赖关系，了解EnglishPal的架构目前的健康水平，并对当前的架构进行优缺点分析。

Introduction
------------

EnglishPal是一款面向希望提升自身英语阅读能力的用户，提供英语文章学习服务的轻量级在线网站。

它。内设了多达百篇不同难度等级的文章，会根据用户的英语水平为用户提供相应难度的文章进行练习；同时，每一个用户还享有独立的生词本，用户可以将在阅读文章过程中遇到的生词收集起来，在复习中进行针对性记忆。

本次实验的目的是通过模块和类及函数两个层面对该项目进行依赖分析，判断模块间、类间的耦合程度以及整体架构的健康水平，为之后的系统维护作准备。

Materials and Methods
---------------------

Materials
~~~~~~~~~

1. Flask: 一个使用 Python 编写的轻量级 Web 应用程序框架。

2. Blueprint: 一个存储视图方法的容器。

3. Httpie: 一个命令行形式的http客户端，提供了简单的http命令，返回带代码高亮的结果信息。

Methods
~~~~~~~

1. 使用Blueprint，在阅读逻辑处理源码、总结出PhotoString项目中各个模块方法的基础上，对原有项目进行模块化处理。

2. 利用httpie，将上传到网页界面的图片信息以json格式展示。

Results
-------

~~~~~~

Lab.py

::

from flask import Flask, redirect, url_for
from show import show_bp
from api import api_bp

app = Flask(__name__)

# 将蓝图注册到app
# 1.注册show蓝图
# 2.注册api蓝图
app.register_blueprint(show_bp, url_prefix="/show")
app.register_blueprint(api_bp)

# 自己本地的项目绝对路径
ch = 'E:/JupyterWork/PhotoString_by_ChenXintao'


# 在运行主界面后，会自动执行此方法
@app.route('/', methods=['POST', 'GET'])
def show():
    # url_for 获取 show_bp.show的url地址
    # redirect 重定向到目标地址
    return redirect(url_for('show_bp.show'))


if __name__ == '__main__':
    app.run()
    app.run(debug=True)

~~~~~~~~~~~~~~~~~~~~~~

将原本Lab中的所有模块功能都细分到了show.py和api.py，并将这两个文件以蓝图形式在Lab.py这个主运行文件中注册。

在主运行文件中，默认路由会重定向到show_bp蓝图中的show方法并执行。

在show.py文件中创建蓝图对象，并命名为“show_bp”

~~~~~~~~~

show.py -> show

::

# show:显示方法
@show_bp.route('/', methods=['GET', 'POST'])
def show():
    # 显示两个表单: 上传表单和检索表单
    # 点击上传表单的submit，跳转到上传的路由执行相应方法
    # 点击检索表单的submit，跳转到检索的路由执行指定条件的检索方法
    page = '''
<form action="/show/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="file" />
    <input name="description" />
    <input type="submit"value="Upload" />
</form>
<form action="/show/search" method="post" enctype="multipart/form-data">
    <input type="text" name="search-str" />
    <input type="submit" value="检索" />
</form>
'''
    # 将数据库中的所有图片及其信息存到page中，输出到页面上
    page += get_database_photos()
    return page


依赖图（Mermaid生成）
~~~~~~~~~~~~~~~~~~~~~

在show.py下的show方法中，在源代码的显示界面的基础上增加了检索的表单。

在程序启动后，网页会默认执行以下方法，将数据库中存储的所有图片以及检索图片功能、上传图片功能显示在网页上。

show.bp 头

::
show_bp = Blueprint("show_bp", __name__)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在show.py文件中创建蓝图对象，并命名为“show_bp”


Discussions
-----------

依赖分析
~~~~~~~~

模块层面
^^^^^^^^

1. wordfreqCMD.py模块

   wordfreqCMD.py模块的作用是转换字符串为列表，并且返回选中单词的使用频率；依赖于wordfreqCMD.py的有WordFreq.py、difficulty.py和main.py

2. WordFreq.py模块

   WordFreq.py模块依赖于wordfreqCMD.py模块，内置方法预处理文章；依赖于WordFreq.py的有main.py

3. difficulty.py模块

   difficulty.py模块同样依赖于wordfreqCMD.py模块，可返回文章的难度值；依赖于difficulty.py的有main.py

4. pickle_idea.py模块和pickle_idea2.py模块

   pickle_idea.py模块依赖于pickle.py，它的作用是存储列表信息，记录单词及其词频,
   
   依赖于pickle_idea.py的有main.py和wordfreqCMD.py；而pickle_idea2.py模块依赖于pickle.py和datetime.py,
   
   它的作用是存储列表信息，记录单词及其日期，依赖于pickle_idea2.py的有main.py

5. UseSqlite.py模块

   UseSqlite.py模块依赖于sqlite3.py,用于连接数据库季相关操作，依赖于UseSqlite.py的有main.py

类及函数层面
^^^^^^^^^^^^

1. Pickle_idea类:

   用于记录单词和词频。被Main类关联，被WordfreqCMD类依赖。
   
   拥有dict2lst(d)，familiar(path, word)，load_record(pickle_fname)，lst2dict(lst, d)，merge_frequency(lst1, lst2)，save_frequency_to_pickle(d,pickle_fname)，unfamiliar(path,word)方法。
   
   其中load_record方法加载pickle文件，save_frequency_to_pickle方法将数据保存到pickle文件中,merge_frequency方法用于合并两个list。
   
2. Pickle_idea2类:

   用于记录单词和日期。被Main类关联。
   
   拥有deleteRecord(path, word)，dict2lst(d)，load_record(pickle_fname)，lst2dict(lst, d)，merge_frequency(lst1, lst2)，save_frequency_to_pickle(d, pickle_fname)方法。
   
3. WordfreqCMD类:
   被Main类关联，被Diffculty类依赖，依赖于Pickle_idea类。拥有file2str(fname)，freq(fruit)，make_html_page(lst, fname)，remove_punctuation(s)，sort_in_ascending_order(lst)，sort_in_descending_order(lst)，youdao_link(s)方法，其中remove_punctuation方法用于删去字符串中的标点。
   
4. Difficulty类:

   被Main类关联，依赖于WordfreqCMD类。拥有difficulty_level_from_frequency(word, d)、get_difficulty_level(d1, d2)、oad_record(pickle_fname)、revert_dict(d)、revert_dict(d)、user_difficulty_level(d_user, d)方法。
   
   其中user_difficulty_level方法用于计算用户等级，其依赖于WordfreqCMD的sort_in_ascending_order方法，text_difficulty_level方法用于计算文章的难度等级，其依赖于WordfreqCMD的freq方法、sort_in_descending_order方法和remove_punctuation方法。
   
5. Main类：

   与Pickle_idea类、Pickle_idea2类、WordfreqCMD类、Difficulty类、Sqlite3Template类关联，拥有add_user(username, password)、appears_in_test(word, d)、check_username_availability(username)、deleteword(username,word)、familiar(username,word)、get_answer_part(s)等方法。

   其中get_today_article方法用于获取文章，该方法依赖于RecordQuery方法，get_difficulty_level方法用于获取读者阅读水平。

6. Sqlite3Template类：

   用于连接数据库。被Main类关联，被InsertQuery类、RecordQuery类继承。拥有__init__(self, db_fname)、connect(self, db_fname)、do(self)、do_with_parameters(self)、format_results(self)、format_results(self)、instructions_with_parameters(self, query_statement, parameters)、operate(self)、operate_with_parameters(self)方法。
   
7. InsertQuery类：用于插入数据。继承Sqlite3Template类。拥有instructions(self, query)
   
8. RecordQuery类：用具记录数据。继承Sqlite3Template类。拥有instructions(self, query)、get_results(self)、instructions(self, query)方法。

当前架构优缺点
~~~~~~~~~~~~~~

优点：
^^^^^^

1. 使用了Flask框架，让项目变得精简、可扩展性强、灵活性高；环境部署简单，只需要在电脑上安装Python的IDE即可在命令行运行。学习完官方文档后可基本了解整体的运行流程，
   易于入门。
   
2. 前后端未被分离，前后端直接进行通信，不产生通信成本。main.py中分模块定义各个功能，通过指定的类和方法去实现对应功能，整体结构较为清晰。

缺点：
^^^^^^

1. 前后端完全混杂，在功能代码中直接进行前端页面编码，嵌入了大量的网页显示代码，严重增加代码阅读难度，也使得后期更新维护难度大大增大。
   
2. 大量类和方法堆叠在main.py内，使得main.py代码长度很长，可适当进行分类存放。

References
----------

[1] `Sofia Peterson, How to Write a Computer Science Lab Report, Copyright (C)
2019 <https://thehackpost.com/a-brief-guide-how-to-write-a-computer-science-lab-report.html>`__

[2] `Georg Brandl and the Sphinx team, reStructuredText Primer Copyright (C)
2007-2021 <https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html>`__

[3] `Martin Blais, Snakefood Copyright (C)
2021 <https://pypi.org/project/snakefood/>`__
