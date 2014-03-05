Lisp 之根本
===========
保罗·格雷厄姆

1960 年，约翰·麦卡锡发表过[一篇非同凡响的论文] [1]，他在其中对程序设计领域的贡献，有如欧几里德对几何学的贡献。他演示了一种方法，仅使用几个简单的操作符和一个表示函数的标记，就构造出一门完整的程序设计语言。他称这门语言为 Lisp，即“List Processing”的缩写。因为他的主要思想之一，就是仅用一种简单的数据结构——_列表（list）_来表示代码和数据。
[1]: "Recursive Functions of Symbolic Expressions and Their Computation by Machine, Part1." _Communication of the ACM_ 3:4, April 1960, pp. 184-195.

值得注意的是，麦卡锡的发现不仅是计算机史上的里程碑，而且是时下程序设计逐渐趋向的一种模式。在我看来，目前为止只有两种真正整洁、一致的程序设计模式：C 语言模式和 Lisp 语言模式。这两者就像两个制高点, 其间则是一片低洼沼泽。随着计算机变得越来越强大，新开发的语言一直在坚定地向 Lisp 模式靠拢。二十年来，新兴程序设计语言的流行配方就是，取 C 语言模式的运算方式，逐渐加上一些 Lisp 语言模式的特性，例如运行时类型和垃圾回收。

在本文中，我将试着用最简单的术语来解释约翰·麦卡锡的发现。重点不仅是领会某人四十年前得出的有趣理论结果，还在于展示程序设计语言的发展方向。Lisp 的不同寻常之处——也就是它决定性的特质——是它由自己编写而成。为了理解麦卡锡所表述的这个特点，我们将追随他的足迹，将他的数学标记转换成能够运行的 Common Lisp 代码。

## 七个原始操作符

首先，我们给_表达式（expression）_下个定义。表达式可以是一个_原子（atom）_，即一串字母（如 foo)，也可以是包在一对小括号中，由空格分隔的，零个或多个表达式组成的_列表（list）_。下面是一些表达式：

	foo
	()
	(foo)
	(foo bar)
	(a b (c) d)

最后一个表达式是含有四个元素的列表，其第三个元素本身又是一个含有一个元素的列表。

在算术中表达式 1 + 1 得出值2. 正确的Lisp表达式也有值.  如果表达式{\it e}得出
值{\it v},我们说{\it e}_ 返回_{\it v}. 下一步我们将定义几种表达式以及它们的返回值.

如果一个表达式是表,我们称第一个元素为_ 操作符_,其余的元素为_ 自变量_.我们将
定义七个原始(从公理的意义上说)操作符: quote,atom,eq,car,cdr,cons,和
cond.

\begin{enumerate}
\item (quote {\it x}) 返回{\it x}.为了可读性我们把(quote {\it x})简记
  为'{\it x}.

\begin{verbatim}
> (quote a)
a
> 'a
a
> (quote (a b c))
(a b c)
\end{verbatim}

\item (atom {\it x})返回原子t如果{\it x}的值是一个原子或是空表,否则返回().  在Lisp中我们
按惯例用原子t表示真, 而用空表表示假.

\begin{verbatim}
> (atom 'a)
t
> (atom '(a b c))
()
> (atom '())
t
\end{verbatim}

既然有了一个自变量需要求值的操作符, 我们可以看一下quote的作用.  通过引
用(quote)一个表,我们避免它被求值.  一个未被引用的表作为自变量传给象
atom这样的操作符将被视为代码:

\begin{verbatim}
> (atom (atom 'a))
t
\end{verbatim}

反之一个被引用的表仅被视为表, 在此例中就是有两个元素的表:

\begin{verbatim}
> (atom '(atom 'a))
()
\end{verbatim}

这与我们在英语中使用引号的方式一致.  {\rm Cambridge}(剑桥)是一个位于麻萨诸塞
州有90000人口的城镇. 而``{\rm Cambridge}''是一个由9个字母组成的单词.

引用看上去可能有点奇怪因为极少有其它语言有类似的概念.  它和Lisp最与众
不同的特征紧密联系:代码和数据由相同的数据结构构成, 而我们用quote操作符
来区分它们.

\item (eq {\it x} {\it y})返回t如果{\it x}和{\it y}的值是同一个原子或都是空表, 否则返回().

\begin{verbatim}
> (eq 'a 'a)
t
> (eq 'a 'b)
()
> (eq '() '())
t
\end{verbatim}

\item (car {\it x})期望{\it x}的值是一个表并且返回{\it x}的第一个元素.

\begin{verbatim}
> (car '(a b c))
a
\end{verbatim}

\item (cdr {\it x})期望{\it x}的值是一个表并且返回{\it x}的第一个元素之后的所有元素.
\begin{verbatim}
> (cdr '(a b c))
(b c)
\end{verbatim}

\item (cons {\it x} {\it y})期望{\it y}的值是一个表并且返回一个新表,它的第一个元素是{\it x}的值, 后
面跟着{\it y}的值的各个元素.

\begin{verbatim}
> (cons 'a '(b c))
(a b c)
> (cons 'a (cons 'b (cons 'c '())))
(a b c)
> (car (cons 'a '(b c)))
a
> (cdr (cons 'a '(b c)))
(b c)
\end{verbatim}
\item (cond (\pone\dots\eone) \dots (\pn\dots\en)) 的求值规则如下. {\it p}表达式依次求值直到有一个
返回t.  如果能找到这样的{\it p}表达式,相应的{\it e}表达式的值作为整个cond表达式的
返回值.

\begin{verbatim}
> (cond ((eq 'a 'b) 'first)
        ((atom 'a)  'second))
second
\end{verbatim}

当表达式以七个原始操作符中的五个开头时,它的自变量总是要求值的.\footnote{以另外两个操作符quote和cond开头的表达式以不同的方式求值. 当
  quote表达式求值时, 它的自变量不被求值,而是作为整个表达式的值返回. 在
  一个正确的cond表达式中, 只有L形路径上的子表达式会被求值.} 我们称这样
  的操作符为_ 函数_. 
\end{enumerate}

\section{函数的表示}
接着我们定义一个记号来描述函数.函数表示为(lambda (\pone\dots\pn) {\it e}),其中
\pone\dots\pn是原子(叫做_ 参数_),{\it e}是表达式. 如果表达式的第一个元素形式如
上

\noindent{\tt
((lambda (\pone\dots\pn) {\it e}) \aone\dots\an)
}

\noindent则称为_ 函数调用_.它的值计算如下.每一个表达式{$a_{i}$}先求值,然后{\it e}再求值.在{\it e}的
求值过程中,每个出现在{\it e}中的{$p_{i}$}的值是相应的{$a_{i}$}在最近一
次的函数调用中的值. 

\begin{verbatim}
> ((lambda (x) (cons x '(b))) 'a)
(a b)
> ((lambda (x y) (cons x (cdr y)))
   'z
   '(a b c))
(z b c)
\end{verbatim}
如果一个表达式的第一个元素{\it f}是原子且{\it f}不是原始操作符

\noindent{\tt
(f \aone\dots\an)
}

\noindent并且{\it f}的值是一个函数(lambda (\pone\dots\pn)),则以上表达式的值就是

\noindent{\tt
((lambda (\pone\dots\pn) {\it e}) \aone\dots\an)
}

\noindent的值.  换句话说,参数在表达式中不但可以作为自变量也可以作为操作符使用:

\begin{verbatim}
> ((lambda (f) (f '(b c)))
   '(lambda (x) (cons 'a x)))
(a b c)
\end{verbatim}

有另外一个函数记号使得函数能提及它本身,这样我们就能方便地定义递归函
数.\footnote{逻辑上我们不需要为了这定义一个新的记号. 在现有的记号中用
  一个叫做Y组合器的函数上的函数, 我们可以定义递归函数. 可能麦卡锡在写
  这篇论文的时候还不知道Y组合器; 无论如何, label可读性更强.} 记号

\noindent{\tt
(label f (lambda (\pone\dots\pn) {\it e}))
}

\noindent表示一个象(lambda (\pone\dots\pn) {\it e})那样的函数,加上这样的特性:
任何出现在{\it e}中的{\it f}将求值为此label表达式, 就好象{\it f}是此函数的参数.

假设我们要定义函数(subst {\it x y z}), 它取表达式{\it x},原子{\it y}和表{\it z}做参数,返回一个
象{\it z}那样的表, 不过{\it z}中出现的{\it y}(在任何嵌套层次上)被{\it x}代替.
\begin{verbatim}
> (subst 'm 'b '(a b (a b c) d))
(a m (a m c) d)
\end{verbatim}
我们可以这样表示此函数
\begin{verbatim}
(label subst (lambda (x y z)
               (cond ((atom z)
                      (cond ((eq z y) x)
                            ('t z)))
                     ('t (cons (subst x y (car z))
                               (subst x y (cdr z)))))))
\end{verbatim}
我们简记{\it f}=(label {\it f} (lambda (\pone\dots\pn) {\it e}))为

\noindent{\tt
(defun {\it f} (\pone\dots\pn) {\it e})
}

\noindent于是
\begin{verbatim}
(defun subst (x y z)
  (cond ((atom z)
         (cond ((eq z y) x)
               ('t z)))
        ('t (cons (subst x y (car z))
                  (subst x y (cdr z))))))
\end{verbatim}
偶然地我们在这儿看到如何写cond表达式的缺省子句. 第一个元素是't的子句总
是会成功的. 于是

\noindent{\tt
(cond ({\it x y}) ('t {\it z}))
}

\noindent等同于我们在某些语言中写的

\noindent{\tt
if {\it x} then {\it y} else {\it z}
}

\section{一些函数}
既然我们有了表示函数的方法,我们根据七个原始操作符来定义一些新的函数.
为了方便我们引进一些常见模式的简记法.  我们用c{\it x}r,其中{\it x}是a或d的序列,来
简记相应的car和cdr的组合. 比如(cadr {\it e})是(car (cdr {\it e}))的简记,它返回{\it e}的
第二个元素.

\begin{verbatim}
> (cadr '((a b) (c d) e))
(c d)
> (caddr '((a b) (c d) e))
e
> (cdar '((a b) (c d) e))
(b)
\end{verbatim}
我们还用(list \eone\dots\en)表示(cons \eone\dots(cons \en '()) \dots).
\begin{verbatim}
> (cons 'a (cons 'b (cons 'c '())))
(a b c)
> (list 'a 'b 'c)
(a b c)
\end{verbatim}

现在我们定义一些新函数. 我在函数名后面加了点,以区别函数和定义它们的原
始函数,也避免与现存的common Lisp的函数冲突.

\begin{enumerate}
\item (null. {\it x})测试它的自变量是否是空表.

\begin{verbatim}
(defun null. (x)
  (eq x '()))

> (null. 'a)
()
> (null. '())
t
\end{verbatim}

\item (and. {\it x y})返回t如果它的两个自变量都是t, 否则返回().

\begin{verbatim}
(defun and. (x y)
  (cond (x (cond (y 't) ('t '())))
        ('t '())))

> (and. (atom 'a) (eq 'a 'a))
t
> (and. (atom 'a) (eq 'a 'b))
()
\end{verbatim}

\item (not. {\it x})返回t如果它的自变量返回(),返回()如果它的自变量返回t.

\begin{verbatim}
(defun not. (x)
  (cond (x '())
        ('t 't)))

> (not. (eq 'a 'a))
()
> (not. (eq 'a 'b))
t
\end{verbatim}

\item (append. {\t x y})取两个表并返回它们的连结.

\begin{verbatim}
(defun append. (x y)
   (cond ((null. x) y)
         ('t (cons (car x) (append. (cdr x) y)))))

> (append. '(a b) '(c d))
(a b c d)
> (append. '() '(c d))
(c d)
\end{verbatim}

\item (pair. {\it x y})取两个相同长度的表,返回一个由双元素表构成的表,双元素表是相
应位置的x,y的元素对.

\begin{verbatim}
(defun pair. (x y)
  (cond ((and. (null. x) (null. y)) '())
        ((and. (not. (atom x)) (not. (atom y)))
         (cons (list (car x) (car y))
               (pair. (cdr) (cdr y))))))

> (pair. '(x y z) '(a b c))
((x a) (y b) (z c))
\end{verbatim}

\item (assoc. {\it x y})取原子{\it x}和形如pair.函数所返回的表{\it y},返回{\it y}中第一个符合如下条
件的表的第二个元素:它的第一个元素是{\it x}.

\begin{verbatim}
(defun assoc. (x y)
  (cond ((eq (caar y) x) (cadar y))
        ('t (assoc. x (cdr y)))))

> (assoc. 'x '((x a) (y b)))
a
> (assoc. 'x '((x new) (x a) (y b)))
new
\end{verbatim}
\end{enumerate}

\section{一个惊喜}
因此我们能够定义函数来连接表,替换表达式等等.也许算是一个优美的表示法,
那下一步呢?  现在惊喜来了. 我们可以写一个函数作为我们语言的解释器:此函
数取任意Lisp表达式作自变量并返回它的值. 如下所示:

\begin{verbatim}
(defun eval. (e a)
  (cond 
    ((atom e) (assoc. e a))
    ((atom (car e))
     (cond 
       ((eq (car e) 'quote) (cadr e))
       ((eq (car e) 'atom)  (atom   (eval. (cadr e) a)))
       ((eq (car e) 'eq)    (eq     (eval. (cadr e) a)
                                    (eval. (caddr e) a)))
       ((eq (car e) 'car)   (car    (eval. (cadr e) a)))
       ((eq (car e) 'cdr)   (cdr    (eval. (cadr e) a)))
       ((eq (car e) 'cons)  (cons   (eval. (cadr e) a)
                                    (eval. (caddr e) a)))
       ((eq (car e) 'cond)  (evcon. (cdr e) a))
       ('t (eval. (cons (assoc. (car e) a)
                        (cdr e))
                  a))))
    ((eq (caar e) 'label)
     (eval. (cons (caddar e) (cdr e))
            (cons (list (cadar e) (car e)) a)))
    ((eq (caar e) 'lambda)
     (eval. (caddar e)
            (append. (pair. (cadar e) (evlis. (cdr  e) a))
                     a)))))

(defun evcon. (c a)
  (cond ((eval. (caar c) a)
         (eval. (cadar c) a))
        ('t (evcon. (cdr c) a))))

(defun evlis. (m a)
  (cond ((null. m) '())
        ('t (cons (eval.  (car m) a)
                  (evlis. (cdr m) a)))))
\end{verbatim}
eval.的定义比我们以前看到的都要长. 让我们考虑它的每一部分是如何工作的.

eval.有两个自变量: e是要求值的表达式, a是由一些赋给原子的值构成的表,这
些值有点象函数调用中的参数.  这个形如pair.的返回值的表叫做_ 环境_.  正是
为了构造和搜索这种表我们才写了pair.和assoc..

eval.的骨架是一个有四个子句的cond表达式.  如何对表达式求值取决于它的类
型. 第一个子句处理原子.  如果e是原子, 我们在环境中寻找它的值:

\begin{verbatim}
> (eval. 'x '((x a) (y b)))
a
\end{verbatim}

第二个子句是另一个cond, 它处理形如({\it a} \dots)的表达式, 其中{\it a}是原子.  这包
括所有的原始操作符, 每个对应一条子句.

\begin{verbatim}
> (eval. '(eq 'a 'a) '())
t
> (eval. '(cons x '(b c))
         '((x a) (y b)))
(a b c)
\end{verbatim}
这几个子句(除了quote)都调用eval.来寻找自变量的值.

最后两个子句更复杂些. 为了求cond表达式的值我们调用了一个叫
evcon.的辅助函数. 它递归地对cond子句进行求值,寻找第一个元素返回t的子句.  如果找到
了这样的子句, 它返回此子句的第二个元素.

\begin{verbatim}
> (eval. '(cond ((atom x) 'atom)
                ('t 'list))
         '((x '(a b))))
list
\end{verbatim}

第二个子句的最后部分处理函数调用. 它把原子替换为它的值(应该是lambda
或label表达式)然后对所得结果表达式求值.  于是

\begin{verbatim}
(eval. '(f '(b c))
       '((f (lambda (x) (cons 'a x)))))
\end{verbatim}
变为
\begin{verbatim}
(eval. '((lambda (x) (cons 'a x)) '(b c))
       '((f (lambda (x) (cons 'a x)))))
\end{verbatim}
它返回(a b c).

eval.的最后cond两个子句处理第一个元素是lambda或label的函数调用.为了对label
表达式求值, 先把函数名和函数本身压入环境, 然后调用eval.对一个内部有
lambda的表达式求值. 即:

\begin{verbatim}
(eval. '((label firstatom (lambda (x)
                            (cond ((atom x) x)
                                  ('t (firstatom (car x))))))
         y)
       '((y ((a b) (c d)))))
\end{verbatim}
变为
\begin{verbatim}
(eval. '((lambda (x)
           (cond ((atom x) x)
                 ('t (firstatom (car x)))))
         y)
        '((firstatom
           (label firstatom (lambda (x)
                            (cond ((atom x) x)
                                  ('t (firstatom (car x)))))))
          (y ((a b) (c d)))))
\end{verbatim}
最终返回a.

最后,对形如((lambda (\pone\dots\pn) {\it e}) \aone\dots\an)的表达式求值,先调用evlis.来
求得自变量(\aone\dots\an)对应的值(\vone\dots\vn),把(\pone \vone)\dots(\pn \vn)添加到
环境里, 然后对{\it e}求值.  于是

\begin{verbatim}
(eval. '((lambda (x y) (cons x (cdr y)))
         'a
         '(b c d))
       '())
\end{verbatim}
变为
\begin{verbatim}
(eval. '(cons x (cdr y))
       '((x a) (y (b c d))))
\end{verbatim}
最终返回(a c d).


\section{后果}

既然理解了eval是如何工作的, 让我们回过头考虑一下这意味着什么. 我们在这
儿得到了一个非常优美的计算模型. 仅用quote,atom,eq,car,cdr,cons,和cond,
我们定义了函数eval.,它事实上实现了我们的语言,用它可以定义任何我们想要
的额外的函数.

当然早已有了各种计算模型---最著名的是图灵机.  但是图灵机程序难以读懂.
如果你要一种描述算法的语言, 你可能需要更抽象的, 而这就是约翰麦卡锡定义
Lisp的目标之一.

约翰麦卡锡于1960年定义的语言还缺不少东西. 它没有副作用, 没有连续执行
(它得和副作用在一起才有用), 没有实际可用的数,\footnote{在麦卡锡的1960
  年的Lisp中, 做算术是可能的, 比如用一个有n个原子的表表示数n.} 没有动态可视域. 但这些限制可
以令人惊讶地用极少的额外代码来补救.  Steele和Sussman在一篇叫做``解释器
的艺术''的著名论文中描述了如何做到这点.\footnote{Guy Lewis
  Steele, Jr. and Gerald Jay Sussman, ``The Art of the Interpreter, or
the Modularity Complex(Parts Zero,One,and Two),'' MIT AL Lab Memo 453,
May 1978.}

如果你理解了约翰麦卡锡的eval, 那你就不仅仅是理解了程序语言历史中的一个
阶段.  这些思想至今仍是Lisp的语义核心.  所以从某种意义上, 学习约翰麦卡
锡的原著向我们展示了Lisp究竟是什么. 与其说Lisp是麦卡锡的设计,不如说是
他的发现. 它不是生来就是一门用于人工智能, 快速原型开发或同等层次任务的
语言. 它是你试图公理化计算的结果(之一). 

随着时间的推移, 中级语言, 即被中间层程序员使用的语言, 正一致地向Lisp靠
近.  因此通过理解eval你正在明白将来的主流计算模式会是什么样.


\section{注释}
把约翰麦卡锡的记号翻译为代码的过程中我尽可能地少做改动.  我有过让代码
更容易阅读的念头, 但是我还是想保持原汁原味.

在约翰麦卡锡的论文中,假用f来表示, 而不是空表. 我用空表表示假以使例子能
在Common Lisp中运行.  (fixme)

我略过了构造dotted pairs, 因为你不需要它来理解eval.  我也没有提apply,
虽然是apply(它的早期形式, 主要作用是引用自变量), 被约翰麦卡锡在1960年
称为普遍函数, eval只是不过是被apply调用的子程序来完成所有的工作.

我定义了list和c{\it x}r等作为简记法因为麦卡锡就是这么做的.  实际上
c{\it x}r等可以
被定义为普通的函数. List也可以这样, 如果我们修改eval, 这很容易做到, 让
函数可以接受任意数目的自变量.

麦卡锡的论文中只有五个原始操作符.  他使用了cond和quote,但可能把它们作
为他的元语言的一部分. 同样他也没有定义逻辑操作符and和not, 这不是个问题,
因为它们可以被定义成合适的函数.

在eval.的定义中我们调用了其它函数如pair.和assoc.,但任何我们用原始操作
符定义的函数调用都可以用eval.来代替.  即 
\begin{verbatim}
(assoc. (car e) a)
\end{verbatim}
能写成

\begin{verbatim}
(eval. '((label assoc.
                (lambda (x y)
                  (cond ((eq (caar y) x) (cadar y))
                        ('t (assoc. x (cdr y))))))
         (car e)
         a)
        (cons (list 'e e) (cons (list 'a a) a)))
\end{verbatim}

麦卡锡的eval有一个错误. 第16行是(相当于)(evlis. (cdr e) a)而不是(cdr
e), 这使得自变量在一个有名函数的调用中被求值两次.  这显示当论文发表的
时候, eval的这种描述还没有用IBM 704机器语言实现.  它还证明了如果不去运
行程序, 要保证不管多短的程序的正确性是多么困难.

我还在麦卡锡的论文中碰到一个问题.  在定义了eval之后, 他继续给出了一些
更高级的函数---接受其它函数作为自变量的函数.  他定义了maplist:

\begin{verbatim}
(label maplist
       (lambda (x f)
         (cond ((null x) '())
               ('t (cons (f x) (maplist (cdr x) f))))))
\end{verbatim}
然后用它写了一个做微分的简单函数diff.  但是diff传给maplist一个用{\it x}做参
数的函数, 对它的引用被maplist中的参数x所捕获.\footnote{当代的Lisp程序
  员在这儿会用mapcar代替maplist.  这个例子解开了一个谜团: maplist为什
  么会在Common Lisp中. 它是最早的映射函数, mapcar是后来增加的.}

这是关于动态可视域危险性的雄辩证据, 即使是最早的更高级函数的例子也因为
它而出错.  可能麦卡锡在1960年还没有充分意识到动态可视域的含意.  动态可
视域令人惊异地在Lisp实现中存在了相当长的时间---直到Sussman和Steele于
1975年开发了Scheme.  词法可视域没使eval的定义复杂多少, 却使编译器更难
写了.

\newpage
\end{CJK*}
\end{document}
