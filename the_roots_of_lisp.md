Lisp 之根本
===========
Paul Graham


1960 年，John McCarthy 发表过一篇非同凡响的论文，他在其中对程序设计领域的贡献，有如欧几里德对几何学的贡献。[<sup>1</sup>](#footnote1) 他演示了一种方法，给定几个简单的操作符和一个表示函数的标记（notation），就可以构造出一门完整的编程语言。他称这门语言为 Lisp，即“List Processing”（列表处理）的缩写，因为其主要思想之一就是，代码和数据皆用同一种叫做_列表（list）_的简单数据结构来承载。

[<a name="footnote1">1</a>]: "Recursive Functions of Symbolic Expressions and Their Computation by Machine, Part1." _Communication of the ACM_ 3:4, April 1960, pp. 184-195.

  McCarthy 的发现值得我们深究，因为它不仅是计算机史上的里程碑，而且是时下程序设计逐渐趋向的一种模式。在我看来，目前为止真正整洁、一致的编程模式只有两种：C 语言模式和 Lisp 语言模式。它们就像两座高峰，其间则遍布低洼沼泽。随着计算机的发展壮大，新开发的语言一直在坚定地向 Lisp 模式靠拢。二十年来，新兴程序设计语言的流行秘方就是，取 C 语言的运算模式，零星点缀一些 Lisp 语言模式的碎片，例如运行时类型和垃圾回收。

  在本文中，我将试着用最简单的术语来解释 John McCarthy 的发现。重点不仅在于领会四十年前某人取得的有趣理论成果，还在于展示编程语言的前进方向。Lisp 的不同寻常之处——也就是它决定性的特质——在于它可以由自己编写而成。为了理解 McCarthy 这句话的含义，我们将追随他的足迹，同时将他的数学标记转换成可执行的 Common Lisp 代码。


## 一、七个基本操作符

首先，我们定义一下_表达式（expression）_。表达式既可以是一个_原子（atom）_，即一串英文字母（如 foo)，也可以是零个或多个表达式组成的_列表（list）_，它们由空格隔开并被包裹在一对小括号中。下面就有一些表达式：

```
foo
()
(foo)
(foo bar)
(a b (c) d)
```

最后一个表达式是个拥有四个元素的列表，而其第三个元素本身又是一个拥有单个元素的列表。

  在算术运算中，表达式 1 + 1 的值为 2。有效的 Lisp 表达式也有值。如果表达式 _e_ 取值为 _v_，我们就说 _e 返回（return） v_。接下来我们将定义都有哪些表达式，以及它们各自的返回值是什么。

  如果某个表达式是列表，我们就称其第一个元素为_操作符（operator）_，其余的元素为_实际参数（argument，以下简称实参）_。我们将定义七个基本（相当于预设的公理）操作符：quote、atom、eq、car、cdr、cons 和 cond。

1. (quote _x_) 返回 _x_。为了增加可读性，我们把 (quote _x_) 简记为 '_x_。

    ```
    > (quote a)
    a
    > 'a
    a
    > (quote (a b c))
    (a b c)
    ```

2. (atom _x_) 如果 _x_ 的值是一个原子或空列表，就返回原子 t，否则返回 ()。在 Lisp 中我们习惯用原子 t 表示真，用空列表表示假。

    ```
    > (atom 'a)
    t
    > (atom '(a b c))
    ()
    > (atom '())
    t
    ```

    有了对实参求值的操作符, 我们就能看到 quote 的作用了。我们通过引用（quote）列表来避免它被求值。如果向类似 atom 这样的操作符传递一个未经引用的列表，它将被视为代码。

    ```
    > (atom (atom 'a))
    t
    ```

    反之，一个被引用的列表仅被当作单纯的列表，在此例中就是拥有两个元素的列表：

    ```
    > (atom '(atom 'a))
    ()
    ```

    这与我们在英语中使用引号的方式一致。Cambridge（剑桥）是一个位于麻省，拥有 9 万人口的城镇，而 "Cambridge" 是一个由 9 个字母组成的单词。

    引用，看上去可能有点陌生，因为几乎没有其他语言里有类似的概念。它和 Lisp 最与众不同的特征之一紧密相连：代码和数据由相同的数据结构构成, 而我们借助 quote 操作符来区分它们。

3. (eq _x_ _y_) 如果 _x_ 和 _y_ 的值是同一个原子或都是空列表, 就返回 t，否则返回 ()。

    ```
    > (eq 'a 'a)
    t
    > (eq 'a 'b)
    ()
    > (eq '() '())
    t
    ```

4. (car _x_) 要求 _x_ 的值是一个列表，并返回该列表的第一个元素。

    ```
    > (car '(a b c))
    a
    ```

5. (cdr _x_) 要求 _x_ 的值是一个列表，并返回该列表第一个元素之后的所有元素。

    ```
    > (cdr '(a b c))
    (b c)
    ```

6. (cons _x_ _y_) 要求 _y_ 的值是一个列表，并返回一个列表，该列表包含 _x_ 的值，后面跟着 _y_ 的值里的所有元素。

    ```
    > (cons 'a '(b c))
    (a b c)
    > (cons 'a (cons 'b (cons 'c '())))
    (a b c)
    > (car (cons 'a '(b c)))
    a
    > (cdr (cons 'a '(b c)))
    (b c)
    ```

7. (cond (_p_<sub>1</sub> _e_<sub>1</sub>) ... (_p_<sub>n</sub> _e_<sub>n</sub>)) 的求值规则如下。对各个 _p_ 表达式依次求值，首次遇到返回 t 的 _p_ 表达式时，就返回其相应的 _e_ 表达式的值作为整个 cond 表达式的值。

    ```
    > (cond ((eq 'a 'b) 'first)
            ((atom 'a)  'second))
    second
    ```

当被求值的表达式以七个基本操作符中的五个开头时，它的实参总是被求值。[<sup>2</sup>](#footnote2) 我们把这类操作符叫做_函数（function）_。

[<a name="footnote2">2</a>]: 以另外两个操作符 quote 和 cond 开头的表达式，其求值方式有所不同。当 quote 表达式被求值时, 它的实参并未被求值，而是作为整个表达式的值返回。在一个正确的 cond 表达式中，只有 L 形支路上的子表达式会被求值。


## 二、函数的表示

接下来，我们定义一个标记来描述函数。函数表示为 (lambda (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_)，其中 _p_<sub>1</sub> ... _p_<sub>n</sub> 都是原子（叫做_形式参数（parameter，以下简称形参）_)，而 _e_ 是表达式。以上述表达式作为第一个元素的那些表达式

((lambda (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_) _a_<sub>1</sub> ... _a_<sub>n</sub>)

被称作一次_函数调用（function call）_，其值按如下方式计算。先对每个 _a_<sub>i</sub> 表达式求值，然后对 _e_ 求值。在 _e_ 的求值过程中，_e_ 中出现的每个 _p_<sub>i</sub> 的值是相应的 _a_<sub>i</sub> 在最近一次函数调用中的值。

```
> ((lambda (x) (cons x '(b))) 'a)
(a b)
> ((lambda (x y) (cons x (cdr y)))
   'z
   '(a b c))
(z b c)
```

如果某个表达式的第一个元素 _f_ 是原子且不是基本操作符

(f _a_<sub>1</sub> ... _a_<sub>n</sub>)

并且 _f_ 的值是一个函数 (lambda (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_)，则上述表达式的值就是

((lambda (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_) _a_<sub>1</sub> ... _a_<sub>n</sub>)

的值。换句话说，在表达式中，形参除了被当作实参，还可作为操作符使用：

```
> ((lambda (f) (f '(b c)))
   '(lambda (x) (cons 'a x)))
(a b c)
```

除此之外，还有一个能让函数引用自己的标记，因而给我们定义递归函数提供了便利。[<sup>3</sup>](#footnote3) 

(label f (lambda (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_))

这一标记所表示的函数，其行为就像是 (lambda (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_)，并附带如下属性，在 _e_ 中出现的任何 _f_ 都将被作为该 label 表达式求值，就好像 _f_ 是函数的形参。

[<a name="footnote3">3</a>]: 逻辑上我们并不需要为此定义一个新标记。我们可以使用现有的标记，借助一种叫做 Y 组合子（Y combinator）的高阶函数来定义递归函数。或许 McCathy 撰写这篇论文时还不知道 Y 组合子。无论如何, label 标记更具可读性。

假设我们要定义一个函数 (subst _x y z_)，它接受一个表达式 _x_，一个原子 _y_ 和一个列表 _z_ 并返回一个像 _z_ 那样的列表，将 _z_ 中出现的 _y_（在任何嵌套层次上）替换为 _x_。

```
> (subst 'm 'b '(a b (a b c) d))
(a m (a m c) d)
```

我们可以将此函数记作

```
(label subst (lambda (x y z)
               (cond ((atom z)
                      (cond ((eq z y) x)
                            ('t z)))
                     ('t (cons (subst x y (car z))
                               (subst x y (cdr z)))))))
```

我们简记 _f_ = (label _f_ (lambda (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_)) 为

(defun _f_ (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_)

于是

```
(defun subst (x y z)
  (cond ((atom z)
         (cond ((eq z y) x)
               ('t z)))
        ('t (cons (subst x y (car z))
                  (subst x y (cdr z))))))
```

我们在这里偶然看到了如何实现 cond 表达式的缺省子句。第一个元素是 't 的子句总是会成功。于是

(cond (_x y_) ('t _z_))

等价于我们使用其他语言中的如下语法

if _x_ then _y_ else _z_


## 三、几个函数

既然有了表示函数的方法，我们就基于七个基本操作符来定义一些新的函数。首先，为了方便起见，我们引进一些常见模式的简记法。我们使用 c_x_r，其中 _x_ 是一系列的 a 或 d，来简记相应的 car 和 cdr 的组合。因此 (cadr _e_) 是 (car (cdr _e_))的简记，它返回 _e_ 的第二个元素。

```
> (cadr '((a b) (c d) e))
(c d)
> (caddr '((a b) (c d) e))
e
> (cdar '((a b) (c d) e))
(b)
```

另外，我们用 (list _e_<sub>1</sub> ... _e_<sub>n</sub>) 表示 (cons _e_<sub>1</sub> ... (cons _e_<sub>n</sub> '()) ... )。

```
> (cons 'a (cons 'b (cons 'c '())))
(a b c)
> (list 'a 'b 'c)
(a b c)
```

   现在我们定义几个新的函数。我在这些函数的名称后面加上了英文句号。这一举措可将基本函数与那些由它们构成的函数区分开，同时也避免了与现存的 Common Lisp 函数冲突。

1. (null. _x_) 测试它的实参是否是空列表。

    ```
    (defun null. (x)
        (eq x '()))

    > (null. 'a)
    ()
    > (null. '())
    t
    ```

2. (and. _x y_) 如果它的两个形参都为 t 就返回 t, 否则返回 ()。

    ```
    (defun and. (x y)
        (cond (x (cond (y 't) ('t '())))
            ('t '())))

    > (and. (atom 'a) (eq 'a 'a))
    t
    > (and. (atom 'a) (eq 'a 'b))
    ()
    ```

3. (not. _x_) 如果它的实参返回 () 就返回 t，反之亦然。

    ```
    (defun not. (x)
        (cond (x '())
            ('t 't)))

    > (not. (eq 'a 'a))
    ()
    > (not. (eq 'a 'b))
    t
    ```

4. (append. _x y_) 接受两个列表并返回它们连接后的结果。

    ```
    (defun append. (x y)
        (cond ((null. x) y)
                ('t (cons (car x) (append. (cdr x) y)))))

    > (append. '(a b) '(c d))
    (a b c d)
    > (append. '() '(c d))
    (c d)
    ```

5. (pair. _x y_) 接受两个相同长度的列表，并返回一个由双元素列表构成的列表，其中的双元素列表是按顺序从两个列表各取一个元素构成的。

    ```
    (defun pair. (x y)
        (cond ((and. (null. x) (null. y)) '())
            ((and. (not. (atom x)) (not. (atom y)))
                (cons (list (car x) (car y))
                    (pair. (cdr x) (cdr y))))))

    > (pair. '(x y z) '(a b c))
    ((x a) (y b) (z c))
    ```

6. (assoc. _x y_) 接受原子 _x_ 和类似 pair. 函数返回值的列表 _y_，并返回 _y_ 中第一个以元素 _x_ 开头的列表的第二个元素。

    ```
    (defun assoc. (x y)
        (cond ((eq (caar y) x) (cadar y))
            ('t (assoc. x (cdr y)))))

    > (assoc. 'x '((x a) (y b)))
    a
    > (assoc. 'x '((x new) (x a) (y b)))
    new
    ```


## 四、惊喜所在

我们已经可以定义函数来实现诸如列表连接，表达式替换等功能了。或许是种优雅的标记法，但也不过如此吧？那么惊喜来了。事实证明，我们还可以编写一个函数来充当我们语言的解释器：此函数接受任意 Lisp 表达式作为实参，并返回它的值。如下所示：

  ```
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
  ```

eval. 的函数定义比我们在前文见过的都长。我们就来考察一下每个部分的作用。

  该函数接受两个实参：e 是即将被求值的表达式，而 a 是一个列表，用于表示在函数调用中作为形参的所有原子的取值。这种形如 pair. 返回值的列表叫做_环境（environment）_。正是为了构造和查找这种列表，我们才编写了 pair. 和 assoc.。

  eval. 的主干是一个带有四个子句的 cond 表达式。我们对表达式求值的方式取决于它们是哪种类型。第一个子句负责处理原子。如果 e 是原子，我们就在环境中查找它的值：

```
> (eval. 'x '((x a) (y b)))
a
```

  eval. 的第二个子句是另一个 cond，它处理形如 (_a_ ...) 的表达式，其中 _a_ 是原子。这里包括所有的原始操作符，每个操作符对应一个子句。

```
> (eval. '(eq 'a 'a) '())
t
> (eval. '(cons x '(b c))
         '((x a) (y b)))
(a b c)
```

这几个子句（除了 quote）都调用 eval. 来寻找实参的值。

  最后两个子句则更复杂些。为了对 cond 表达式求值，我们调用了一个叫做 evcon. 的辅助函数，该函数递归寻找满足如下条件的子句：该子句的第一个元素返回 t。当它找到这样的子句，就返回其第二个元素的值。

```
> (eval. '(cond ((atom x) 'atom)
                ('t 'list))
         '((x '(a b))))
list
```

  eval. 第二个子句的结尾部分负责处理那些作为形参传进来的函数的调用。它把原子替换为对应的值（应该是 lambda 或 label 表达式）再对得到的结果表达式求值。于是

```
(eval. '(f '(b c))
       '((f (lambda (x) (cons 'a x)))))
```

变为

```
(eval. '((lambda (x) (cons 'a x)) '(b c))
       '((f (lambda (x) (cons 'a x)))))
```

它返回(a b c)。

eval. 的最后两个子句处理那些刚好以 lambda 或 label 表达式开头的函数调用。对 label 表达式求值，要先把由函数名和函数本身组成的列表压入环境，然后调用 eval. 对剥开 label 表达式得到的 lambda 表达式求值，即：

```
(eval. '((label firstatom (lambda (x)
                            (cond ((atom x) x)
                                  ('t (firstatom (car x))))))
         y)
       '((y ((a b) (c d)))))
```

变为

```
(eval. '((lambda (x)
           (cond ((atom x) x)
                 ('t (firstatom (car x)))))
         y)
        '((firstatom
           (label firstatom (lambda (x)
                            (cond ((atom x) x)
                                  ('t (firstatom (car x)))))))
          (y ((a b) (c d)))))
```

最终返回a.

最后，对形如 ((lambda (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_) _a_<sub>1</sub> ... _a_<sub>n</sub>) 的表达式求值，要先调用 evlis. 
求得实参 (_a_<sub>1</sub> ... _a_<sub>n</sub>) 相应取值的列表 (_v_<sub>1</sub> ... _v_<sub>n</sub>)，然后把 (_p_<sub>1</sub> _v_<sub>1</sub>) ... (_p_<sub>n</sub> _v_<sub>n</sub>) 添加到
环境的开头，并对 _e_ 求值。于是

```
(eval. '((lambda (x y) (cons x (cdr y)))
         'a
         '(b c d))
       '())
```

变为

```
(eval. '(cons x (cdr y))
       '((x a) (y (b c d))))
```

最终返回(a c d)。


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

我定义了list和c_ x_r等作为简记法因为麦卡锡就是这么做的.  实际上
c_ x_r等可以
被定义为普通的函数. List也可以这样, 如果我们修改eval, 这很容易做到, 让
函数可以接受任意数目的自变量.

麦卡锡的论文中只有五个原始操作符.  他使用了cond和quote,但可能把它们作
为他的元语言的一部分. 同样他也没有定义逻辑操作符and和not, 这不是个问题,
因为它们可以被定义成合适的函数.

在eval.的定义中我们调用了其它函数如pair.和assoc.,但任何我们用原始操作
符定义的函数调用都可以用eval.来代替.  即 
```
(assoc. (car e) a)
```
能写成

```
(eval. '((label assoc.
                (lambda (x y)
                  (cond ((eq (caar y) x) (cadar y))
                        ('t (assoc. x (cdr y))))))
         (car e)
         a)
        (cons (list 'e e) (cons (list 'a a) a)))
```

麦卡锡的eval有一个错误. 第16行是(相当于)(evlis. (cdr e) a)而不是(cdr
e), 这使得自变量在一个有名函数的调用中被求值两次.  这显示当论文发表的
时候, eval的这种描述还没有用IBM 704机器语言实现.  它还证明了如果不去运
行程序, 要保证不管多短的程序的正确性是多么困难.

我还在麦卡锡的论文中碰到一个问题.  在定义了eval之后, 他继续给出了一些
更高级的函数---接受其它函数作为自变量的函数.  他定义了maplist:

```
(label maplist
       (lambda (x f)
         (cond ((null x) '())
               ('t (cons (f x) (maplist (cdr x) f))))))
```
然后用它写了一个做微分的简单函数diff.  但是diff传给maplist一个用_ x_做参
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
