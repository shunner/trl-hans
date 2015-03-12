Lisp 之根本
===========
Paul Graham


1960 年，John McCarthy 发表了一篇引人注目的论文，他在文中对程序设计领域的贡献，如同欧几里德对几何学的贡献。[<sup>1</sup>](#footnote1) 他演示了如何通过给定几个简单的操作符和一个表示函数的标记（notation）就可以构造出一门完整的编程语言。他称这门语言为 Lisp，即“List Processing（列表处理）”，因为他的核心思想之一就是把一种叫做_列表（list）_的简单数据结构同时用于代码和数据。

[<a name="footnote1">1</a>]: "Recursive Functions of Symbolic Expressions and Their Computation by Machine, Part I." _Communication of the ACM_ 3:4, April 1960, pp. 184-195.[<sup>译注1</sup>](#commentary1)

  McCarthy 的发现非常值得深入理解，因为它不仅是计算机史上的里程碑，更是程序设计当下趋向的一种模型。在我看来，目前为止真正整洁、一致的编程模型只有两种：C 语言模型和 Lisp 语言模型。它们就像两座高峰，其间则遍布低洼沼泽。随着计算机变得更加强大，新开发的语言一直坚定地向 Lisp 模型靠拢。二十年来，新兴程序设计语言的流行秘方就是取 C 语言的计算模型，零星点缀一些 Lisp 模型的碎片，例如运行时类型（runtime typing）和垃圾回收（garbage collection）。

  在本文中，我将试着用最简单的术语来解释 McCarthy 的发现。重点不仅在于领会某人在四十年前取得的有趣理论成果，还在于展示编程语言的发展方向。Lisp 的不同寻常之处——其实是它决定性的特质——是它可以用其自身编写而成。为了理解 McCarthy 这句话的含义，我们将循着他的足迹，将他的数学标记转换成可运行的 Common Lisp 代码。


## 一、七操作符

首先，我们定义一下_表达式（expression）_。表达式既可以是一个_原子（atom）_，即一串英文字母（如 foo)，也可以是包含零个或多个表达式的_列表（list）_，其中的表达式由空格隔开，并被包裹在一对小括号中。下面这些都是表达式：

```
foo
()
(foo)
(foo bar)
(a b (c) d)
```

最后的那个表达式是个含有四个元素的列表，而该列表的第三个元素本身又是个含有单一元素的列表。

  在算术运算中，表达式 1 + 1 的值为 2。有效的 Lisp 表达式也有值。如果表达式 _e_ 取值为 _v_，我们就说 _e 返回（return） v_。下一步我们将定义表达式都有哪些类型，以及它们各自返回什么值。

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

  2. (atom _x_) 如果 _x_ 的值是一个原子或空列表，就返回原子 t，否则返回 ()。在 Lisp 中我们习惯用原子 t 代表真，用空列表来代表假。

    ```
    > (atom 'a)
    t
    > (atom '(a b c))
    ()
    > (atom '())
    t
    ```

    既然我们有了对实参求值的操作符，就可以展现 quote 的作用了。我们通过引用（quote）列表来避免它被求值。当一个未经引用的列表被作为实参传递给类似 atom 这样的操作符时，即被视为代码。

    ```
    > (atom (atom 'a))
    t
    ```

    反之，一个被引用的列表仅被当作单纯的列表，在此例中就是个含有两个元素的列表：

    ```
    > (atom '(atom 'a))
    ()
    ```

    这与我们在英语中使用引号的方式一致。Cambridge（坎布里奇）是一个位于麻省，拥有 9 万人口的城镇，"Cambridge" 则是一个包含 9 个字母的单词。[<sup>译注</sup>](#commentary)

    引用，乍一看可能有点陌生，因为几乎没什么语言有类似的概念。它和 Lisp 最与众不同的特征之一紧密相连：代码和数据由相同的数据结构实现，而 quote 操作符正是我们借以区分二者的手段。

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

  5. (cdr _x_) 要求 _x_ 的值是一个列表，并返回该列表第一个元素之后的全部元素。

    ```
    > (cdr '(a b c))
    (b c)
    ```

  6. (cons _x_ _y_) 要求 _y_ 的值是一个列表，并返回一个列表，其中包含 _x_ 的值，随后是 _y_ 的值里的所有元素。

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

  7. (cond (_p_<sub>1</sub> _e_<sub>1</sub>) ... (_p_<sub>n</sub> _e_<sub>n</sub>)) 的求值规则如下。对各个 _p_ 表达式依次求值，首次遇到返回 t 的 _p_ 表达式时，就返回相应 _e_ 表达式的值作为整个 cond 表达式的值。

    ```
    > (cond ((eq 'a 'b) 'first)
            ((atom 'a)  'second))
    second
    ```

当被求值的表达式以七个基本操作符中的五个开头时，它的实参总会被求值。[<sup>2</sup>](#footnote2) 我们把这类操作符叫做_函数（function）_。

[<a name="footnote2">2</a>]: 以另外两个操作符 quote 和 cond 开头的表达式，其求值方式有所不同。当 quote 表达式被求值时, 它的实参并未被求值，而是作为整个表达式的值返回。在一个正确的 cond 表达式中，只有 L 形支路上的子表达式会被求值。


## 二、函数表示

接下来，我们定义一个标记来描述函数。函数用 (lambda (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_) 来表示，其中 _p_<sub>1</sub> ... _p_<sub>n</sub> 都是原子（叫做_形式参数（parameter，以下简称形参）_），而 _e_ 是表达式。以上述表达式作为第一个元素的那些表达式

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

的值。换句话说，形参在表达式中既可用作实参，也可用作操作符：

```
> ((lambda (f) (f '(b c)))
   '(lambda (x) (cons 'a x)))
(a b c)
```

还有另外一个标记能让函数引用自己，从而给我们定义递归函数提供了便利。[<sup>3</sup>](#footnote3) 

(label f (lambda (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_))

这一标记所表示的函数，其行为就像是 (lambda (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_)，并附带如下属性，在 _e_ 中出现的任何 _f_ 都将被当作该 label 表达式求值，如同 _f_ 是函数的形参。

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

我们在这里顺便看到了如何编写 cond 表达式的缺省子句。第一个元素是 't 的子句总是会成功。于是

(cond (_x y_) ('t _z_))

等价于我们用支持如下语法的语言编写的代码

if _x_ then _y_ else _z_


## 三、几个函数

既然有了表示函数的方法，我们就基于七个基本操作符来定义一些新的函数。首先，为了方便起见，我们引进一些常见模式的简记法。我们使用 c_x_r，其中 _x_ 是一系列的 a 或 d，来简记相应的 car 和 cdr 的组合。因此 (cadr _e_) 是 (car (cdr _e_)) 的简记，它返回 _e_ 的第二个元素。

```
> (cadr '((a b) (c d) e))
(c d)
> (caddr '((a b) (c d) e))
e
> (cdar '((a b) (c d) e))
(b)
```

此外，我们用 (list _e_<sub>1</sub> ... _e_<sub>n</sub>) 表示 (cons _e_<sub>1</sub> ... (cons _e_<sub>n</sub> '()) ... )。

```
> (cons 'a (cons 'b (cons 'c '())))
(a b c)
> (list 'a 'b 'c)
(a b c)
```

  现在我们定义几个新函数。我在这些函数的名称后面加上了英文句号。这样就把基本函数与那些由它们构成的函数区分开来，同时也避免了与现存的 Common Lisp 函数冲突。

  1. (null. _x_) 测试它的实参是否是空列表。

    ```
    (defun null. (x)
      (eq x '()))

    > (null. 'a)
    ()
    > (null. '())
    t
    ```

  2. (and. _x y_) 如果它的两个实参都为 t 就返回 t, 否则返回 ()。

    ```
    (defun and. (x y)
      (cond (x (cond (y 't) ('t '())))
            ('t '())))

    > (and. (atom 'a) (eq 'a 'a))
    t
    > (and. (atom 'a) (eq 'a 'b))
    ()
    ```

  3. (not. _x_) 如果它的实参返回 () 它就返回 t，如果它的实参返回 t 它就返回 ()。

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

至此我们可以定义函数来连接列表、替换表达式等等。或许是个优雅的标记，但也不过如此吧？那么惊喜来了。事实证明，我们还可以编写一个函数来充当我们语言的解释器：此函数接受任意 Lisp 表达式作为实参，并返回它的值。如下所示：

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

eval. 的定义比我们在前文见过的都长。我们就来考察一下每个部分的作用。

  该函数接受两个实参：e 是即将被求值的表达式，而 a 是一个列表，用于表示在函数调用中作为形参的所有原子的取值。这种形如 pair. 返回值的列表叫做_环境（environment）_。正是为了构造和查找这种列表，我们才编写了 pair. 和 assoc.。

  eval. 的主干是一个带有四个子句的 cond 表达式。我们对表达式求值的方式取决于它们是哪种类型。第一个子句负责处理原子。如果 e 是原子，我们就在环境中查找它的值：

```
> (eval. 'x '((x a) (y b)))
a
```

  eval. 的第二个子句是另一个 cond，它处理形如 (_a_ ...) 的表达式，其中 _a_ 是原子。这里包括全部的基本操作符，每个操作符对应一个子句。

```
> (eval. '(eq 'a 'a) '())
t
> (eval. '(cons x '(b c))
         '((x a) (y b)))
(a b c)
```

这几个子句（除了 quote）都调用 eval. 来寻找实参的值。

  最后两个子句则更复杂些。为了对 cond 表达式求值，我们调用了一个叫做 evcon. 的辅助函数，该函数递归寻找这样的子句：该子句的第一个元素返回 t。当它找到这样的子句，就返回其第二个元素的值。

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

最终返回 a.

  最后，对形如 ((lambda (_p_<sub>1</sub> ... _p_<sub>n</sub>) _e_) _a_<sub>1</sub> ... _a_<sub>n</sub>) 的表达式求值，要先调用 evlis. 求得实参 (_a_<sub>1</sub> ... _a_<sub>n</sub>) 相应取值组成的列表 (_v_<sub>1</sub> ... _v_<sub>n</sub>)，然后把 (_p_<sub>1</sub> _v_<sub>1</sub>) ... (_p_<sub>n</sub> _v_<sub>n</sub>) 添加到环境的开头，并对 _e_ 求值。于是

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

最终返回 (a c d)。


## 五、余音绕梁

既然理解了 eval 的工作原理，就让我们回过头来考虑一下这意味着什么。我们得到的是一个非常简约的计算模型。仅用 quote、atom、eq、car、cdr、cons 和 cond 就可以定义函数 eval.，从而真正实现了我们的语言，而后我们就可以随心所欲地用该语言来定义更多函数。

  计算模型早已有之，最著名的当属图灵机。但图灵机程序读来缺乏启发性。如果你想要一门用来描述算法的语言，可能需要它更加抽象，而这正是 McCarthy 定义 Lisp 的目标之一。

  他在 1960 年定义的这门语言缺少很多特性。它没有函数副作用（side-effect），没有连续执行（sequential execution）（尽管它仅在函数副作用存在时才有用），没有实际数（practical
number），[<sup>4</sup>](#footnote4)没有动态作用域（dynamic scope）。但这些限制可以用少得惊人的附加代码来补救。Steele 和 Sussman 在一篇著名的论文 The Art of the Interpreter（《解释器的艺术》）中描述了如何做到这点。[<sup>5</sup>](#footnote5)

[<a name="footnote4">4</a>]: 在 McCarthy 1960 年版的 Lisp 中是可以进行算术运算的，比如用一个有 _n_ 个原子的列表来表示 _n_ 这个数。

[<a name="footnote5">5</a>]: Guy Lewis Steele, Jr. and Gerald Jay Sussman, "The Art of the Interpreter, or the Modularity Complex (Parts Zero, One, and Two)," MIT AI Lab Memo 453, May 1978.

  如果你理解了 McCarthy 的 eval，你所理解的就远不止编程语言的一个历史阶段。这些思想至今仍是 Lisp 的语义核心。因此在某种意义上，学习 McCarthy 的原著向我们展示了 Lisp 的本来面目。它并不是 McCarthy 的设计，而是他的发现。它并不是一门专用于人工智能、快速原型开发或同一级别的其他任务的语言。它是你试图对计算进行公理化时所得到的结果(或至少是之一)。

  随着时间的推移，中级（median）语言，即中级程序员所使用的语言，始终在向 Lisp 靠拢。因此通过理解 eval 你始终能够很好地理解主流计算模式将会是什么。


## 后记

在把 McCarthy 的标记翻译为可运行代码的过程中，我尽可能少做改动。我曾有过让代码更易读的念头, 但还是想保持原汁原味。

  在 McCarthy 的论文中，假值是用 f 而不是空列表来表示的。我用 () 表示假值，以便让示例能在 Common Lisp 中正常运行。没有一处代码依赖于假值也恰好是空列表；没有任何内容被添加（cons）到谓词返回的结果中。

  我跳过了使用点对（dotted pair）构造列表的过程，因为你不需要借此理解 eval。我也没有提及 apply，虽然正是这个 apply（它的早期形式，主要作用是引用实参）被 McCarthy 在 1960 年称为万能（universal）函数；而那时的 eval 只是被 apply 调用的一段子程序，用来完成所有工作。

  我定义 list 和 c_x_r 作为简记法是源于 McCarthy 的做法。实际上 c_x_r 本来都可以被定义为普通的函数。而 list 亦如是，只要我们简单修改下 eval，让函数可以接受任意数目的实参即可。

  McCarthy 的论文中仅有五个基本操作符。虽然他使用了 cond 和 quote，但多半是将其作为元语言（metalanguage）的一部分。他同样没有定义逻辑操作符 and 和 not，不过这不成问题，因为它们可以被恰如其分地定义为函数。

  在 eval. 的定义中我们调用了其他函数，如 pair. 和 assoc.，但我们用基本操作符定义的任一函数都可以替换为 eval. 来调用。亦即

```
(assoc. (car e) a)
```

其实可以写作

```
(eval. '((label assoc.
                (lambda (x y)
                  (cond ((eq (caar y) x) (cadar y))
                        ('t (assoc. x (cdr y))))))
         (car e)
         a)
        (cons (list 'e e) (cons (list 'a a) a)))
```

  McCarthy 的 eval 有一个小缺陷。第 16 行（相当于）是 (evlis. (cdr e) a) 而不是 (cdr e)，导致命名函数的实参在一次调用中被求值了两次。这表明论文发表时，这段对 eval 的描述还没有用 IBM 704 机器语言实现过。它还证明了未曾尝试运行就要确保程序的正确性是多么困难，无论程序长短。

  我在 McCarthy 的代码中还碰到了另一个问题。在定义了 eval 之后，他继续给出了一些高阶函数（higher-order function）——接受其它函数作为实参的函数——的例子。他定义了 maplist：

```
(label maplist
       (lambda (x f)
         (cond ((null x) '())
               ('t (cons (f x) (maplist (cdr x) f))))))
```

然后用它写了一个用于符号微分（symbolic differentiation）的简单函数 diff。而 diff 向 maplist 传递一个以 x 为形参
的函数，对其的引用却被 maplist 内的形参 x 所捕获。[<sup>6</sup>](#footnote6)

[<a name="footnote6">6</a>]: 当代 Lisp 程序员在这儿会用 mapcar 代替 maplist。这个例子解开了一个谜： 为什么在 Common Lisp 中会有 maplist。它是最早的映射（mapping）函数，而 mapcar 是后来增加的。

这是关于动态作用域危险性的雄辩证据，即使是最早的 Lisp 高阶函数的例子也因为它而出错。可能 McCarthy 在 1960 年还没有充分意识到动态作用域的后果。动态作用域令人惊异地在 Lisp 的多种实现中存在了相当长的时间——直到 Sussman 和 Steele 于 1975 年开发了 Scheme。词法作用域（lexcical scope）并没让 eval 的定义复杂多少，却使编译器更加难以编写。

[<a name="commentary1">译注1</a>]: 论文的题目表明它只是 Part I，如果读者也好奇 Part II 是什么内容的话，McCarthy 在[他的个人主页](http://www-formal.stanford.edu/jmc/#proglang)做了解答：Part II 从未面世，原本是要给出一些进行代数运算（algrebaic computation）的 Lisp 程序。

[<a name="commentary">译注</a>]: 位于美国马萨诸塞州（又称麻省）的 Cambridge 因与英国剑桥同名，常被译作“美国剑桥”或“坎布里奇”。此地正是 McCarthy 当时供职的 MIT（麻省理工学院）之所在。
