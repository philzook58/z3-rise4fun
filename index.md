---
title: "Getting Started with Z3: A Guide"
---

<script src="/out/z3.js" ></script>
<script>
   function run_id(code_id,result_id){
      const code = document.getElementById(code_id);
      const result = document.getElementById(result_id);
      result.innerText = Z3.solve(code.value)
   }
</script>




<textarea id="ex0_code" rows="20" style="width:100%">
   (check-sat)
</textarea><br/>
<button onClick="run_id('ex0_code','ex0_result')" >Solve</button> <br/>
<code id="ex0_result" ></code>



<p style="clear:both;">Be sure to follow along with the examples by clicking the "edit" link in the
   corner. See what the tool says, try your own formulas, and experiment!</p>
   
   <h2>Introduction</h2>
   
   <p>
   Z3 is a state-of-the art theorem prover from Microsoft Research. It  
   can be used to check the satisfiability of logical formulas over one or more theories. 
   Z3 offers a compelling match for software analysis and verification tools, since several 
   common software constructs map directly into supported theories.
   </p>
   
   <p>
   The main objective of the tutorial is to introduce the reader on how to use
   Z3 effectively for logical modeling and solving.
   The tutorial provides some general background on logical modeling, 
   but we have to defer a full introduction to first-order logic and 
   decision procedures to text-books.
   </p>
   
   <p>
   Z3 is a low level tool. It is best used
   as a component in the context of other tools that require solving logical
   formulas. Consequently, Z3 exposes a number of API facilities to make it 
   convenient for tools to map into Z3, but there are no stand-alone 
   editors or user-centric facilities for interacting with Z3. 
   The language syntax used in the front ends favor simplicity in contrast
   to linguistic convenience. 
   </p>
   
   <h2>Basic Commands</h2>
   
   <p>
   The Z3 input format is an extension of the one defined by the 
   <a href="http://www.smtlib.org/" target="_blank">SMT-LIB 2.0 standard</a>.
   A Z3 script is a sequence of commands. 
   The <b>help</b> command displays a list of all available
   commands. The command <b>echo</b> displays a message. 
   Internally, Z3 maintains a <b>stack</b>
   of user provided formulas and declarations. 
   We say these are the <b>assertions</b> provided by the user. 
   The command <b>declare-const</b> declares a constant of a given type (aka sort). 
   The command <b>declare-fun</b> declares a function. 
   In the following example, we declared a function that
   receives an integer and a boolean, and returns an integer.
   </p>
   <textarea id="ex1_code" rows="3" style="width:100%">
      (echo "starting Z3...")
      (declare-const a Int)
      (declare-fun f (Int Bool) Int)
   </textarea><br/>
   <button id="ex1" onClick="run_id('ex1_code','ex1_result')" >Solve</button> <br/>
   <code id="ex1_result" ></code>
   <p>
   The command <b>assert</b>
   adds a formula into the Z3 internal stack. 
   We say the set of formulas in the Z3 stack is <b>satisfiable</b>
   if there is an interpretation (for the user declared constants and functions)
   that makes all asserted formulas true.
   </p>
   <textarea id="ex2_code" rows="20" style="width:100%">
      (declare-const a Int)
      (declare-fun f (Int Bool) Int)
      (assert (> a 10))
      (assert (< (f a true) 100))
      (check-sat)
   </textarea><br/>
   <button onClick="run_id('ex2_code','ex2_result')" >Solve</button> <br/>
   <code id="ex2_result" ></code>
   <p>
   The first asserted formula states that the constant <tt>a</tt> must be greater than 10.
   The second one states that the function <tt>f</tt> applied to <tt>a</tt> and <tt>true</tt> must
   return a value less than 100.
   The command <b>check-sat</b> determines whether the current formulas on the Z3 stack
   are satisfiable or not. 
   If the formulas are satisfiable, Z3 returns <b>sat</b>. If they are not satisfiable (i.e., they are <b>unsatisfiable</b>),
   Z3 returns <b>unsat</b>. Z3 may also return <b>unknown</b> when it can't determine whether a formula
   is satisfiable or not.
   </p>
   
   <p> 
   When the command check-sat returns <b>sat</b>, the command <b>get-model</b> can be used to retrieve 
   an interpretation that makes all formulas on the Z3 internal stack true.
   </p> 
   <textarea id="ex3_code" rows="20" style="width:100%">
      (declare-const a Int)
      (declare-fun f (Int Bool) Int)
      (assert (&gt; a 10))
      (assert (&lt; (f a true) 100))
      (check-sat)
      (get-model)
   </textarea><br/>
   <button onClick="run_id('ex3_code','ex3_result')" >Solve</button> <br/>
   <code id="ex3_result" ></code>
   
   <p>
   The interpretation is provided using <b>definitions</b>. 
   For example, the definition: </p>
   <pre class="listing">
   (define-fun a () Int [val])
   </pre>
   <p>
   states that the value of <tt>a</tt> in the model is <tt>[val]</tt>.
   The definition: </p>
   <pre class="listing">
   (define-fun f ((x!1 Int) (x!2 Bool)) Int
      ...
   )
   </pre>
   <p>
   is very similar to a function definition used in programming languages.
   In this example, <tt>x1</tt> and <tt>x2</tt> are the arguments of the function interpretation
   created by Z3. For this simple example, the definition of <tt>f</tt> is based on
   <b>ite</b>'s (aka if-then-elses or conditional expressions). For example, the expression:
   </p>
   <pre class="listing">
   (ite (and (= x!1 11) (= x!2 false)) 21 0)
   </pre>
   <p>
   evaluates (returns) 21 when <tt>x!1</tt> is equal to 11, and <tt>x!2</tt> is equal to false.
   Otherwise, it returns 0.
   </p>
   
   <h3>Using Scopes</h3>
   
   <p>
   In some applications, we want to explore several similar problems that share several definitions and assertions.
   We can use the commands <b>push</b> and <b>pop</b> for doing that. Z3 maintains a global stack of declarations
   and assertions. The command <tt>push</tt> creates a new scope by saving the current stack size. 
   The command <tt>pop</tt> removes any assertion or declaration performed between it and the matching push.
   The <tt>check-sat</tt> and <tt>get-assertions</tt> commands always operate on the content of the global stack.
   </p>
   
   <p>In the following example, the command <tt>(assert p)</tt> signs an error because the <tt>pop</tt> command 
   removed the declaration for <tt>p</tt>. If the last <tt>pop</tt> command is removed, then the error is corrected.
   </p>
   <textarea id="ex4_code" rows="20" style="width:100%">
      (declare-const x Int)
      (declare-const y Int)
      (declare-const z Int)
      (push)
      (assert (= (+ x y) 10))
      (assert (= (+ x (* 2 y)) 20))
      (check-sat)
      (pop) ; remove the two assertions
      (push) 
      (assert (= (+ (* 3 x) y) 10))
      (assert (= (+ (* 2 x) (* 2 y)) 21))
      (check-sat)
      (declare-const p Bool)
      (pop)
      (assert p) ; error, since declaration of p was removed from the stack   
   </textarea><br/>
   <button onClick="run_id('ex4_code','ex4_result')" >Solve</button> <br/>
   <code id="ex4_result" ></code>

   
   <p>The <tt>push</tt> and <tt>pop</tt> commands can optionally receive
   a numeral argument as specifed by the SMT 2 language.
   </p>
   
   <h3>Configuration</h3>
   
   <p>The command <b>set-option</b> is used to configure Z3. Z3 has several options to control its behavior.
   Some of these options (e.g., <tt>:produce-proofs</tt>) can only be set before any declaration or assertion.
   We use the <b>reset</b> command to erase all assertions and declarations. After the <tt>reset</tt> command, all 
   configuration options can be set.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/0nSM?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (set-option :print-success true) 
   (set-option :produce-unsat-cores true) ; enable generation of unsat cores
   (set-option :produce-models true) ; enable model generation
   (set-option :produce-proofs true) ; enable proof generation
   (declare-const x Int)
   (set-option :produce-proofs false) ; error, cannot change this option after a declaration or assertion
   (echo "before reset")
   (reset)
   (set-option :produce-proofs false) ; ok
   </pre>
   
   <p>
   The option <tt>:print-success true</tt> is particularly useful when Z3 is being controlled by another application using pipes.
   In this mode, commands, that otherwise would not print any output, will print <tt>success</tt>.
   </p>
   
   <h3>Additional commands</h3>
   
   <p>The command <b>(display t)</b> just applies the Z3 pretty printer to the given expression.
   The command <b>(simplify t)</b> displays a possibly simpler expression equivalent to <tt>t</tt>.
   This command accepts many different options, <tt>(help simplify)</tt> will display all available options.
   </p>
   <textarea id="ex6_code" rows="20" style="width:100%">
      (declare-const a (Array Int Int))
      (declare-const x Int)
      (declare-const y Int)
      (display (+ x 2 x 1))
      (simplify (+ x 2 x 1))
      (simplify (* (+ x y) (+ x y)))
      (simplify (* (+ x y) (+ x y)) :som true) ; put all expressions in sum-of-monomials form.
      (simplify (= x (+ y 2)) :arith-lhs true)
      (simplify (= (store (store a 1 2) 4 3)
                   (store (store a 4 3) 1 2)))
      (simplify (= (store (store a 1 2) 4 3)
                   (store (store a 4 3) 1 2))
                :sort-store true)
      (help simplify)
   </textarea><br/>
   <button onClick="run_id('ex6_code','ex6_result')" >Solve</button> <br/>
   <code id="ex6_result" ></code>
   
   <p>
   The <b>define-sort</b> command defines a new sort symbol that is an abbreviation for a sort expression.
   The new sort symbol can be parameterized, in which case the names of the parameters
   are specified in the command and the sort expression uses the sort parameters. The form of the
   command is this:
   </p>
   
   <pre class="listing">
   (define-sort [symbol] ([symbol]+) [sort])
   </pre>
   
   <p>
   The following example defines several abbreviations for sort expressions.
   </p>
   <textarea id="ex7_code" rows="20" style="width:100%">
      (define-sort Set (T) (Array T Bool))
      (define-sort IList () (List Int))
      (define-sort List-Set (T) (Array (List T) Bool))
      (define-sort I () Int)
      
      (declare-const s1 (Set I))
      (declare-const s2 (List-Set Int))
      (declare-const a I)
      (declare-const l IList)
      
      (assert (= (select s1 a) true))
      (assert (= (select s2 l) false))
      (check-sat)
      (get-model)
   </textarea><br/>
   <button onClick="run_id('ex7_code','ex7_result')" >Solve</button> <br/>
   <code id="ex7_result" ></code>
   
   <h2>Propositional Logic</h2>
   
   <p>
   The pre-defined sort <bb>Bool</bb> is the sort (type) of all Boolean propositional expressions.
   Z3 supports the usual Boolean operators <tt>and</tt>, <tt>or</tt>, <tt>xor</tt>, <tt>not</tt>, <tt>=&gt;</tt> (implication), <tt>ite</tt> (if-then-else).
   Bi-implications are represented using equatity <tt>=</tt>.
   The following example shows how to prove that if <tt>p</tt> implies <tt>q</tt> and <tt>q</tt> implies <tt>r</tt>, then <tt>p</tt> implies <tt>r</tt>.
   We accomplish that by showing that the negation is unsatisfiable. The command <bb>define-fun</bb> is used to define a macro (aka alias). 
   In this example, <tt>conjecture</tt> is an alias for the conjecture we want to prove.
   </p>
   <textarea id="ex8_code" rows="20" style="width:100%">
      (declare-const p Bool)
      (declare-const q Bool)
      (declare-const r Bool)
      (define-fun conjecture () Bool
         (=&gt; (and (=&gt; p q) (=&gt; q r))
            (=&gt; p r)))
      (assert (not conjecture))
      (check-sat)
   </textarea><br/>
   <button onClick="run_id('ex8_code','ex8_result')" >Solve</button> <br/>
   <code id="ex8_result" ></code>

   
   <h3>Satisfiability and Validity</h3>
   
   <p>A formula <tt>F</tt> is <b>valid</b> if <tt>F</tt> always evaluates to true for any assignment of appropriate values to its
   uninterpreted function and constant symbols. 
   A formula <tt>F</tt> is <b>satisfiable</b> if there is some assignment of appropriate values
   to its uninterpreted function and constant symbols under which <tt>F</tt> evaluates to true. 
   Validity is about finding a proof of a statement; satisfiability is about finding a solution to a set of constraints.
   Consider a formula <tt>F</tt> with some uninterpreted constants, say <tt>a</tt> and <tt>b</tt>. 
   We can ask whether <tt>F</tt> is valid, that is whether it is always true for any combination of values for 
   <tt>a</tt> and <tt>b</tt>. If <tt>F</tt> is always
   true, then <tt>not F</tt> is always false, and then <tt>not F</tt> will not have any satisfying assignment; that is, 
   <tt>not F</tt> is unsatisfiable. That is, 
   <tt>F</tt> is valid precisely when <tt>not F</tt> is not satisfiable (is unsatisfiable).
   Alternately,
   <tt>F</tt> is satisfiable if and only if <tt>not F</tt> is not valid (is invalid).
   Z3 finds satisfying assignments (or report that there are none). To determine whether a
   formula <tt>F</tt> is valid, we ask Z3 whether <tt>not F</tt> is satisfiable. 
   Thus, to check the deMorgan's law is valid (i.e., to prove it), we show its negation to be unsatisfiable.</p>
   <textarea id="ex9_code" rows="20" style="width:100%">
      (declare-const a Bool)
      (declare-const b Bool)
      (define-fun demorgan () Bool
          (= (and a b) (not (or (not a) (not b)))))
      (assert (not demorgan))
      (check-sat)
   </textarea><br/>
   <button onClick="run_id('ex9_code','ex9_result')" >Solve</button> <br/>
   <code id="ex9_result" ></code>

   
   <h2>Uninterpreted functions and constants</h2>
   
   <p>
   The basic building blocks of SMT formulas are constants and  
   functions. Constants are just functions that take no
   arguments.
   So everything is really just a function.
   </p>
   <textarea id="ex10_code" rows="20" style="width:100%">
      (declare-fun f (Int) Int)
      (declare-fun a () Int) ; a is a constant
      (declare-const b Int) ; syntax sugar for (declare-fun b () Int)
      (assert (&gt; a 20))
      (assert (&gt; b a))
      (assert (= (f 10) 1))
      (check-sat)
      (get-model)
   </textarea><br/>
   <button onClick="run_id('ex10_code','ex10_result')" >Solve</button> <br/>
   <code id="ex10_result" ></code>
   
   <p>
   Unlike programming languages, where functions have side-effects, can throw exceptions, 
   or never return, functions in classical first-order logic have no side-effects and are <b>total</b>.
   That is, they are defined on all input values. This includes functions, such
   as division.
   </p>
   
   <p>
   Function and constant symbols in pure first-order logic are <i>uninterpreted</i> or <i>free</i>, 
   which means that no a priori interpretation is attached.
   This is in contrast to functions belonging to the signature of theories,
   such as arithmetic where the function <tt>+</tt> has a fixed standard interpretation
   (it adds two numbers). Uninterpreted functions and constants are maximally flexible;
   they allow any interpretation that is consistent with the constraints over the function or constant.
   </p>
   
   <p>
   To illustrate uninterpreted functions and constants let us introduce an (uninterpreted) sort
   <tt>A</tt>, and the constants <tt>x</tt>, <tt>y</tt> ranging over <tt>A</tt>. Finally let <tt>f</tt>
   be an uninterpreted function that takes one argument of sort <tt>A</tt> and results in a value
   of sort <tt>A</tt>. The example illustrates how one can force an interpretation where <tt>f</tt>
   applied twice to <tt>x</tt> results in <tt>x</tt> again, but <tt>f</tt> applied once to <tt>x</tt> is different form <tt>x</tt>.
   </p>
   
   <textarea id="ex11_code" rows="20" style="width:100%">
      (declare-sort A)
      (declare-const x A)
      (declare-const y A)
      (declare-fun f (A) A)
      (assert (= (f (f x)) x))
      (assert (= (f x) y))
      (assert (not (= x y)))
      (check-sat)
      (get-model)
   </textarea><br/>
   <button onClick="run_id('ex11_code','ex11_result')" >Solve</button> <br/>
   <code id="ex11_result" ></code>

   
   <p>
   The resulting model introduces abstract values for the elements in <tt>A</tt>, because the sort <tt>A</tt>
   is uninterpreted. The interpretation for <tt>f</tt> in the model toggles between the two values for <tt>x</tt>
   and <tt>y</tt>, which are different. 
   </p>
   
   <h2>Arithmetic</h2>
   
   <p>
   Z3 has builtin support for integer and real constants.
   This two types should not be confused with machine integers (32-bit or 64-bit)
   and floating point numbers. These two types (sorts) represent the mathematical
   integers and reals. 
   The command <b>declare-const</b> is used to declare integer and real constants.
   </p>
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/jw?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-const a Int)
   (declare-const b Int)
   (declare-const c Int)
   (declare-const d Real)
   (declare-const e Real)
   </pre>
   
   <p>
   After constants are declared, the user can assert.smt formulas containing these constants.
   The formulas contain arithmetic operators such as: <tt>+</tt>, <tt>-</tt>, <tt>&lt;</tt>, and so on.
   The command <b>check-sat</b> will instruct Z3 to try to find an interpretation for the declared constants
   that makes all formulas true. The interpretation is basically assigning a number to each constant.
   If such interpretation exists, we say it is a <b>model</b> for the asserted formulas. 
   The command <b>get-model</b> displays the model built by Z3.
   </p>
   <textarea id="ex12_code" rows="20" style="width:100%">
      (declare-const a Int)
      (declare-const b Int)
      (declare-const c Int)
      (declare-const d Real)
      (declare-const e Real)
      (assert (&gt; a (+ b 2)))
      (assert (= a (+ (* 2 c) 10)))
      (assert (&lt;= (+ c b) 1000))
      (assert (&gt;= d e))
      (check-sat)
      (get-model)
   </textarea><br/>
   <button onClick="run_id('ex12_code','ex12_result')" >Solve</button> <br/>
   <code id="ex12_result" ></code>
   
   <p>
   Real constants should contain a decimal point. Unlike
   most programming languages, Z3 will not convert automatically integers into reals and
   vice-versa. The function <b>to-real</b> can be used to convert an integer
   expression into a real one.
   </p>
   <textarea id="ex13_code" rows="20" style="width:100%">
      (declare-const a Int)
      (declare-const b Int)
      (declare-const c Int)
      (declare-const d Real)
      (declare-const e Real)
      (assert (&gt; e (+ (to_real (+ a b)) 2.0)))
      (assert (= d (+ (to_real c) 0.5)))
      (assert (&gt; a b))
      (check-sat)
      (get-model)
   </textarea><br/>
   <button onClick="run_id('ex13_code','ex13_result')" >Solve</button> <br/>
   <code id="ex13_result" ></code>

   
   <h3>Nonlinear arithmetic</h3>
   
   <p>
   We say a formula is <b>nonlinear</b> if it contains expressions of the form
   <b>(* t s)</b> where <tt>t</tt> and <tt>s</tt> are not numbers.
   Nonlinear real arithmetic is very expensive, and Z3 is not complete for this kind of formula.
   The command <b>check-sat</b> may return <b>unknown</b> or loop.
   Nonlinear integer arithmetic is <b>undecidable</b>: there is no procedure
   that is correct and terminates (for every input) with a <b>sat</b> or <b>unsat</b> answer.
   Yes, it is <b>impossible</b> to build such procedure. Note that, this does not prevent
   Z3 from returning an answer for many nonlinear problems. The real limit is that there will always be
   a nonlinear integer arithmetic formula that it will fail produce an answer.
   </p>
   <textarea id="ex14_code" rows="20" style="width:100%">
      (declare-const a Int)
      (assert (&gt; (* a a) 3))
      (check-sat)
      (get-model)
      
      (echo "Z3 does not always find solutions to non-linear problems")
      (declare-const b Real)
      (declare-const c Real)
      (assert (= (+ (* b b b) (* b c)) 3.0))
      (check-sat)
      
      (echo "yet it can show unsatisfiabiltiy for some nontrivial nonlinear problems...")
      (declare-const x Real)
      (declare-const y Real)
      (declare-const z Real)
      (assert (= (* x x) (+ x 2.0)))
      (assert (= (* x y) x))
      (assert (= (* (- y 1.0) z) 1.0))
      (check-sat)
      
      (reset)
      (echo "When presented only non-linear constraints over reals, Z3 uses a specialized complete solver")
      (declare-const b Real)
      (declare-const c Real)
      (assert (= (+ (* b b b) (* b c)) 3.0))
      (check-sat)
      (get-model)
   </textarea><br/>
   <button onClick="run_id('ex14_code','ex14_result')" >Solve</button> <br/>
   <code id="ex14_result" ></code>
   
   <h3>Division</h3>
   
   <p>
   Z3 also has support for <b>division</b>, integer division, modulo and remainder operators.
   Internally, they are all mapped to multiplication.  
   </p>
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/0Msl?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-const a Int)
   (declare-const r1 Int)
   (declare-const r2 Int)
   (declare-const r3 Int)
   (declare-const r4 Int)
   (declare-const r5 Int)
   (declare-const r6 Int)
   (assert (= a 10))
   (assert (= r1 (div a 4))) ; integer division
   (assert (= r2 (mod a 4))) ; mod
   (assert (= r3 (rem a 4))) ; remainder
   (assert (= r4 (div a (- 4)))) ; integer division
   (assert (= r5 (mod a (- 4)))) ; mod
   (assert (= r6 (rem a (- 4)))) ; remainder
   (declare-const b Real)
   (declare-const c Real)
   (assert (&gt;= b (/ c 3.0)))
   (assert (&gt;= c 20.0))
   (check-sat)
   (get-model)
   </pre>
   
   <p>
   In Z3, <b>division by zero</b> is allowed,
   but the result is not specified. Division is not a partial function. Actually, in Z3 all
   functions are total, although the result may be underspecified in some cases like division by
   zero.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/ajKV?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-const a Real)
   ; The following formula is satisfiable since division by zero is not specified.
   (assert (= (/ a 0.0) 10.0)) 
   (check-sat)
   (get-model)
   
   ; Although division by zero is not specified, division is still a function.
   ; So, (/ a 0.0) cannot evaluated to 10.0 and 2.0.
   (assert (= (/ a 0.0) 2.0)) 
   (check-sat)
   </pre>
   
   <p>
    If you are not happy with this behavior, you may use <b>ite</b> (if-then-else) operator to guard
   every division, and assign whatever intepretation you like to the division by zero.
   This example uses <b>define-fun</b> constructor to create a new operator: <b>mydiv</b>.
   This is essentially a macro, and Z3 will expand its definition for every application of <b>mydiv</b>. 
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/m?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   ; defining my own division operator where x/0.0 == 0.0 for every x.
   (define-fun mydiv ((x Real) (y Real)) Real
     (if (not (= y 0.0))
         (/ x y)
         0.0))
   (declare-const a Real)
   (declare-const b Real)
   (assert (&gt;= (mydiv a b) 1.0))
   (assert (= b 0.0))
   (check-sat)
   </pre>
   
   <h2>Bitvectors</h2>
   
   <p>
   Modern CPUs and main-stream programming languages use 
   arithmetic over fixed-size bit-vectors.
   The theory of bit-vectors allows modeling the 
   precise semantics of unsigned and of 
   signed two-complements arithmetic. There are a large number 
   of supported functions and relations
   over bit-vectors. They are summarized on Z3's 
   <a href="https://web.archive.org/web/20210125011020/http://research.microsoft.com/projects/z3/documentation.html" target="_blank">on-line documentation</a>
   of the binary APIs and they are summarized on the <a href="https://web.archive.org/web/20210125011020/http://www.smtlib.org/" target="_blank">SMT-LIB</a> web-site.
   We will not try to give a comprehensive overview here, 
   but touch on some of the main features.
   </p>
   
   <p>
   In contrast to programming languages, such as C, C++, C#, Java, 
   there is no distinction between signed and unsigned bit-vectors
   as numbers. Instead, the theory of bit-vectors provides special signed versions of arithmetical operations
   where it makes a difference whether the bit-vector is treated as signed or unsigned.
   </p>
   
   <p>
   Z3 supports Bitvectors of arbitrary size. <tt>(_ BitVec n)</tt> is the sort of bitvectors
   whose length is <tt>n</tt>. Bitvector literals may be defined using binary, decimal and hexadecimal notation.
   In the binary and hexadecimal cases, the bitvector size is inferred from the number of characters.
   For example, the bitvector literal <tt>#b010</tt> in binary format is a bitvector of size 3,
   and the bitvector literal <tt>#x0a0</tt> in hexadecimal format is a bitvector of size 12.
   The size must be specified for bitvector literals in decimal format. For example,
   <tt>(_ bv10 32)</tt> is a bitvector of size 32 that representes the numeral <tt>10</tt>.
   By default, Z3 display bitvectors in hexadecimal format if the bitvector size is a multiple of 4, and in binary otherwise. 
   The command <tt>(set-option :pp.bv-literals false)</tt> can be used to force Z3 to display bitvector literals in
   decimal format.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/8dj7?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (display #b0100)
   (display (_ bv20 8))
   (display (_ bv20 7))
   (display #x0a) 
   (set-option :pp.bv-literals false)
   (display #b0100)
   (display (_ bv20 8))
   (display (_ bv20 7))
   (display #x0a) 
   </pre>
   
   <h3>Basic Bitvector Arithmetic</h3>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/Y?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (simplify (bvadd #x07 #x03)) ; addition
   (simplify (bvsub #x07 #x03)) ; subtraction
   (simplify (bvneg #x07)) ; unary minus
   (simplify (bvmul #x07 #x03)) ; multiplication
   (simplify (bvurem #x07 #x03)) ; unsigned remainder
   (simplify (bvsrem #x07 #x03)) ; signed remainder
   (simplify (bvsmod #x07 #x03)) ; signed modulo
   (simplify (bvshl #x07 #x03)) ; shift left
   (simplify (bvlshr #xf0 #x03)) ; unsigned (logical) shift right
   (simplify (bvashr #xf0 #x03)) ; signed (arithmetical) shift right
   </pre>
   
   <h3>Bitwise Operations</h3>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/6ehA?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing"> 
   (simplify (bvor #x6 #x3))   ; bitwise or
   (simplify (bvand #x6 #x3))  ; bitwise and
   (simplify (bvnot #x6)) ; bitwise not
   (simplify (bvnand #x6 #x3)) ; bitwise nand
   (simplify (bvnor #x6 #x3)) ; bitwise nor
   (simplify (bvxnor #x6 #x3)) ; bitwise xnor
   </pre>
   
   <p>
   We can prove a bitwise version of deMorgan's law:
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/x?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-const x (_ BitVec 64))
   (declare-const y (_ BitVec 64))
   (assert (not (= (bvand (bvnot x) (bvnot y)) (bvnot (bvor x y)))))
   (check-sat)
   </pre>
   
   <p>
   Let us illustrate a simple property of bit-wise arithmetic.
   There is a fast way to check that fixed size numbers are powers of
   two. It turns out that a bit-vector <tt>x</tt> is a power of two or 
   zero if and only if <tt>x &amp; (x - 1)</tt> is zero, where <tt>&amp;</tt> represents the bitwise and.
   We check this for four bits below.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/G?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (define-fun is-power-of-two ((x (_ BitVec 4))) Bool 
     (= #x0 (bvand x (bvsub x #x1))))
   (declare-const a (_ BitVec 4))
   (assert 
    (not (= (is-power-of-two a) 
            (or (= a #x0) 
                (= a #x1) 
                (= a #x2) 
                (= a #x4) 
                (= a #x8)))))
   (check-sat)
   </pre>
   
   <h3>Predicates over Bitvectors</h3>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/r?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (simplify (bvule #x0a #xf0))  ; unsigned less or equal
   (simplify (bvult #x0a #xf0))  ; unsigned less than
   (simplify (bvuge #x0a #xf0))  ; unsigned greater or equal
   (simplify (bvugt #x0a #xf0))  ; unsigned greater than
   (simplify (bvsle #x0a #xf0))  ; signed less or equal
   (simplify (bvslt #x0a #xf0))  ; signed less than
   (simplify (bvsge #x0a #xf0))  ; signed greater or equal
   (simplify (bvsgt #x0a #xf0))  ; signed greater than
   </pre>
   
   <p>Signed comparison, such as <tt>bvsle</tt>, takes the sign bit of bitvectors into account for comparison,
   while unsigned comparison treats the bitvector as unsigned (treats the bitvector as a natural number).
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/sl?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-const a (_ BitVec 4))
   (declare-const b (_ BitVec 4))
   (assert (not (= (bvule a b) (bvsle a b))))
   (check-sat)
   (get-model)
   </pre>
   
   <h2>Arrays</h2>
   
   <p>As part of formulating a programme of a mathematical theory of computation
   McCarthy proposed a <i>basic</i> theory of arrays as characterized by
   the select-store axioms. The expression <tt>(select a i)</tt> returns
   the value stored at position <tt>i</tt> of the array <tt>a</tt>;
   and <tt>(store a i v)</tt> returns a new array identical to <tt>a</tt>,
   but on position <tt>i</tt> it contains the value <tt>v</tt>.
   </p>
   
   <p>
   Z3 contains a decision procedure for the basic theory of
   arrays. By default, Z3 assumes that arrays are
   extensional over select. In other words, Z3 also enforces that if two 
   arrays agree on all reads, then the arrays are equal.
   </p>
   
   <p>
   It also contains various extensions 
   for operations on arrays that remain decidable and amenable
   to efficient saturation procedures (here efficient means, 
   with an NP-complete satisfiability complexity).
   We describe these extensions in the following using a collection of examples.
   Additional background on these extensions is available in the 
   paper <a href="https://web.archive.org/web/20210125011020/http://research.microsoft.com/en-us/um/people/leonardo/fmcad09.pdf" target="_blank">Generalized and Efficient Array Decision Procedures</a>.
   </p>
   
   <h3>Select and Store</h3>
   
   <p>Let us first check a basic property of arrays.
   Suppose <tt>a1</tt> is an array of integers, then the constraint
   <tt>(and (= (select a1 x) x) (= (store a1 x y) a1))</tt>
   is satisfiable for an array that contains an index <tt>x</tt> that maps to <tt>x</tt>,
   and when <tt>x = y</tt> (because the first equality forced the range of <tt>x</tt> to be <tt>x</tt>).
   We can check this constraint.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/iO?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-const x Int)
   (declare-const y Int)
   (declare-const z Int)
   (declare-const a1 (Array Int Int))
   (declare-const a2 (Array Int Int))
   (declare-const a3 (Array Int Int))
   (assert (= (select a1 x) x))
   (assert (= (store a1 x y) a1))
   (check-sat)
   (get-model)
   </pre>
   
   <p>On the other hand, the constraints become unsatisfiable when asserting
   <tt>(not (= x y))</tt>.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/9oKM?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-const x Int)
   (declare-const y Int)
   (declare-const z Int)
   (declare-const a1 (Array Int Int))
   (declare-const a2 (Array Int Int))
   (declare-const a3 (Array Int Int))
   (assert (= (select a1 x) x))
   (assert (= (store a1 x y) a1))
   (assert (not (= x y)))
   (check-sat)
   </pre>
   
   <h3>Constant Arrays</h3>
   
   <p>
   The array that maps all indices to some fixed value can be specified in Z3 using the
   <tt>const</tt> construct. It takes one value from the range type of
   the array and creates an array. Z3 cannot infer what kind of array must be returned
   by the <tt>const</tt> construct by just inspecting the argument type.
   Thus, a qualified identifier <tt>(as const (Array T1 T2))</tt> must be used.
   The following example defines a constant array containing only ones.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/Ii?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-const all1 (Array Int Int))
   (declare-const a Int)
   (declare-const i Int)
   (assert (= all1 ((as const (Array Int Int)) 1)))
   (assert (= a (select all1 i)))
   (check-sat)
   (get-model)
   (assert (not (= a 1)))
   (check-sat)
   </pre>
   
   <h3>Array models</h3>
   
   <p>
   Models provide interpretations of the uninterpreted (aka free) constants  and functions that appear in the satisfiable formula. 
   An interpretation for arrays is very similar to the interpretation of a function.
   Z3 uses the construct <tt>(_ as-array f)</tt> to give the interpretation for arrays.
   If the array <tt>a</tt> is equal to <tt>(_ as-array f)</tt>, then for every index <tt>i</tt>, 
   <tt>(select a i)</tt> is equal to <tt>(f i)</tt>.
   In the previous example, Z3 creates the auxiliary function <tt>k!0</tt> to assign an interpretation to the array constant <tt>all1</tt>.
   </p>
   
   <h3>Mapping Functions on Arrays</h3>
   
   <p>
   In the following, we will simulate basic Boolean algebra
   (set theory) using the array theory extensions in Z3.
   Z3 provides a parametrized <tt>map</tt> function on arrays. 
   It allows applying arbitrary functions to the range of arrays.
   The following example illustrates how to use the <tt>map</tt> function.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/slc?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (define-sort Set (T) (Array T Bool))
   (declare-const a (Set Int))
   (declare-const b (Set Int))
   (declare-const c (Set Int))
   (declare-const x Int)
   (push)
   (assert (not (= ((_ map and) a b) ((_ map not) ((_ map or) ((_ map not) b) ((_ map not) a))))))
   (check-sat)
   (pop)
   (push) 
   (assert (and (select ((_ map and) a b) x) (not (select a x))))
   (check-sat)
   (pop)
   (push) 
   (assert (and (select ((_ map or) a b) x) (not (select a x))))
   (check-sat)
   (get-model)
   (assert (and (not (select b x))))
   (check-sat)
   </pre>
   
   <h3>Bags as Arrays</h3>
   
   <p>
   We can use the parametrized map function to encode
   finite sets and finite bags. Finite bags can be modeled similarly to sets.
   A bag is here an array that maps elements to their multiplicity. 
   Main bag operations include <tt>union</tt>, obtained by adding multiplicity, 
   <tt>intersection</tt>, by taking the minimum multiplicity, and a dual <tt>join</tt>
   operation that takes the maximum multiplicity. In the following example,
   we define the </tt>bag-union</tt> using <tt>map</tt>. Notice that we need
   to specify the full signature of <tt>+</tt> since it is an overloaded operator.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/rN?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (define-sort A () (Array Int Int Int))
   (define-fun bag-union ((x A) (y A)) A
     ((_ map (+ (Int Int) Int)) x y))
   (declare-const s1 A)
   (declare-const s2 A)
   (declare-const s3 A)
   (assert (= s3 (bag-union s1 s2)))
   (assert (= (select s1 0 0) 5))
   (assert (= (select s2 0 0) 3))
   (assert (= (select s2 1 2) 4))
   (check-sat)
   (get-model)
   </pre>
   
   <h2>Datatypes</h2>
   
   <p>
   Algebraic datatypes, known from programming languages such as ML, offer a convenient
   way for specifying common data structures. Records and tuples are special cases of
   algebraic datatypes, and so are scalars (enumeration types). But algebraic datatypes are
   more general. They can be used to specify finite lists, trees and other
   recursive structures. 
   </p>
   
   <h3>Records</h3>
   
   <p>A record is specified as a datatype with a single constructor and as many arguments
   as record elements. The number of arguments to a record are always the same. 
   The type system does not allow to extend records and there is no record subtyping. 
   </p>
   
   <p>
   The following example illustrates that two records are equal only if all the arguments are equal. 
   It introduces the parametric type <tt>Pair</tt>, with constructor <tt>mk-pair</tt>
   and two arguments that can be accessed using the selector functions <tt>first</tt> and <tt>second</tt>.
   </p>
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/LA?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-datatypes (T1 T2) ((Pair (mk-pair (first T1) (second T2)))))
   (declare-const p1 (Pair Int Int))
   (declare-const p2 (Pair Int Int))
   (assert (= p1 p2))
   (assert (&gt; (second p1) 20))
   (check-sat)
   (get-model)
   (assert (not (= (first p1) (first p2))))
   (check-sat)
   </pre>
   
   <h3>Scalars (enumeration types)</h3>
   
   <p>A scalar sort is a finite domain sort. The elements of the finite domain are enumerated
   as distinct constants. For example, the sort <tt>S</tt> is a scalar type with
   three values <tt>A</tt>, <tt>B</tt> and <tt>C</tt>. It is possible for three constants
   of sort <tt>S</tt> to be distinct, but not for four constants.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/IA?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-datatypes () ((S A B C)))
   (declare-const x S)
   (declare-const y S)
   (declare-const z S)
   (declare-const u S)
   (assert (distinct x y z))
   (check-sat)
   (assert (distinct x y z u))
   (check-sat)
   </pre>
   
   <h3>Recursive datatypes</h3>
   
   <p>
   A recursive datatype declaration includes itself directly (or indirectly) as a component.
   A standard example of a recursive data-type is the one of lists. A parametric list
   can be specified in Z3 as:
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/775?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-datatypes (T) ((Lst nil (cons (hd T) (tl Lst)))))
   (declare-const l1 (Lst Int))
   (declare-const l2 (Lst Bool))
   </pre>
   
   <p>The <tt>List</tt> recursive datatype is builtin in Z3.
   The empty list is <tt>nil</tt>, and the constructor <tt>insert</tt> is used to build new lists.
   The accessors <tt>head</tt> and <tt>tail</tt> are defined as usual.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/p?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-const l1 (List Int))
   (declare-const l2 (List Int))
   (declare-const l3 (List Int))
   (declare-const x Int)
   (assert (not (= l1 nil)))
   (assert (not (= l2 nil)))
   (assert (= (head l1) (head l2)))
   (assert (not (= l1 l2)))
   (assert (= l3 (insert x l2)))
   (assert (&gt; x 100))
   (check-sat)
   (get-model)
   (assert (= (tail l1) (tail l2)))
   (check-sat)
   </pre>
   
   <p>In the example above, we also assert that <tt>l1</tt> and <tt>l2</tt> are not <tt>nil</tt>. 
   This is because the interpretation
   of <tt>head</tt> and <tt>tail</tt> is underspecified on <tt>nil</tt>. So then 
   head and tail would not be able to distinguish 
   <tt>nil</tt> from <tt>(insert (head nil) (tail nil))</tt>.
   </p>
   
   <h3>Mutually recursive datatypes</h3>
   
   <p>
   You can also specify mutually recursive datatypes for Z3.
   We list one example below.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/eV?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   ; declare a mutually recursive parametric datatype
   (declare-datatypes (T) ((Tree leaf (node (value T) (children TreeList)))
                           (TreeList nil (cons (car Tree) (cdr TreeList)))))
   (declare-const t1 (Tree Int))
   (declare-const t2 (Tree Bool))
   ; we must use the 'as' construct to distinguish the leaf (Tree Int) from leaf (Tree Bool)
   (assert (not (= t1 (as leaf (Tree Int)))))
   (assert (&gt; (value t1) 20))
   (assert (not (is-leaf t2)))
   (assert (not (value t2)))
   (check-sat)
   (get-model)
   </pre>
   
   <p>In the example above, we have a tree of Booleans and a tree of integers. The <tt>leaf</tt>
   constant must return a tree of a specific sort. To specify the result sort, we use the qualified 
   identifier <tt>(as leaf (Tree Int))</tt>. Note that, we do not need to use a qualified identifer
   for <tt>value</tt>, since Z3 can infer the intended declaration using the sort of the argument.
   </p>
   
   <h3>Z3 will not prove inductive facts</h3>
   
   <p>
   The ground decision procedures for recursive datatypes don't lift
   to establishing inductive facts. Z3 does not contain methods
   for producing proofs by induction. This may change in the future.
   In particular, consider
   the following example where the function <tt>p</tt> is true
   on all natural numbers, which can be proved by induction over
   <tt>Nat</tt>. Z3 enters a matching loop as it attempts instantiating
   the universally quantified implication. 
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/dRWWo?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (set-option :timeout 2000)
   (declare-datatypes () ((Nat zero (succ (pred Nat)))))
   (declare-fun p (Nat) Bool)
   (assert (p zero))
   (assert (forall ((x Nat)) (implies (p (pred x)) (p x))))
   (assert (not (forall ((x Nat)) (p x))))
   (check-sat)
   (get-info :all-statistics)
   </pre>
   
   <h2>Quantifiers</h2>
   
   <p>
   Z3 is a <i>decision procedure</i>
   for the combination of the previous quantifier-free theories.
   That is, it can answer whether a quantifier-free formula, modulo
   the theories referenced by the formula, is satisfiable or whether
   it is unsatisfiable. Z3 also accepts and can work with formulas
   that use quantifiers. It is no longer a decision procedure for 
   such formulas in general (and for good reasons, as there can be
   no decision procedure for first-order logic).
   </p>
   
   <p>
   Nevertheless, Z3 is often able to handle formulas involving
   quantifiers. It uses several approaches to handle quantifiers.
   The most prolific approach is using <i>pattern-based</i> quantifier
   instantiation. This approach allows instantiating quantified formulas
   with ground terms that appear in the current search context based
   on <i>pattern annotations</i> on quantifiers. Another approach
   is based on <i>saturation theorem proving</i> using a superposition
   calculus which is a modern method for applying resolution style 
   rules with equalities.
   The pattern-based instantiation method is quite effective, 
   even though it is inherently incomplete. The saturation based approach
   is complete for pure first-order formulas, but does not scale as 
   nicely and is harder to predict.
   </p>
   
   <p>
   Z3 also contains a model-based quantifier instantiation 
   component that uses a model construction to find good terms to instantiate
   quantifiers with; and Z3 also handles many decidable fragments.
   </p>
   
   <h3>Modeling with Quantifiers</h3>
   
   <p>
   Suppose we want to model an object oriented type system with 
   single inheritance. We would need a predicate for sub-typing.
   Sub-typing should be a partial order, and respect single inheritance.
   For some built-in type constructors, such as for <tt>array-of</tt>, sub-typing
   should be monotone. 
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/pl?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-sort Type)
   (declare-fun subtype (Type Type) Bool)
   (declare-fun array-of (Type) Type)
   (assert (forall ((x Type)) (subtype x x)))
   (assert (forall ((x Type) (y Type) (z Type))
             (=&gt; (and (subtype x y) (subtype y z)) 
                 (subtype x z)))) 
   (assert (forall ((x Type) (y Type))
             (=&gt; (and (subtype x y) (subtype y x)) 
                 (= x y))))
   (assert (forall ((x Type) (y Type) (z Type))
             (=&gt; (and (subtype x y) (subtype x z)) 
                 (or (subtype y z) (subtype z y))))) 
   (assert (forall ((x Type) (y Type))
             (=&gt; (subtype x y) 
                 (subtype (array-of x) (array-of y)))))
   (declare-const root-type Type)
   (assert (forall ((x Type)) (subtype x root-type)))
   (check-sat)
   </pre>
   
   <h3>Patterns</h3>
   
   <p>
   The Stanford Pascal verifier and the subsequent 
   Simplify theorem prover 
   pioneered the use of 
   pattern-based quantifier instantiation. 
   The basic idea behind pattern-based quantifier
   instantiation is in a sense straight-forward:
   Annotate a quantified formula using a <i>pattern</i> that contains
   all the bound variables. 
   So a pattern is an expression (that does not contain binding operations,
   such as quantifiers) that contains variables bound by a 
   quantifier.
   Then instantiate the quantifier
   whenever a term that matches the pattern is created during search.
   This is a conceptually easy starting point, but there are several subtleties
   that are important. 
   </p>
   
   <p>In the following example, 
   the first two options make sure that Model-based quantifier instantiation and saturation engines are disabled.
   We also annotate the quantified formula with the pattern <tt>(f (g x))</tt>.
   Since there is no ground instance of this pattern, the quantifier is not instantiated, and Z3 fails to show that
   the formula is unsatisfiable.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/K?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (set-option :smt.auto-config false) ; disable automatic self configuration
   (set-option :smt.mbqi false) ; disable model-based quantifier instantiation
   (declare-fun f (Int) Int)
   (declare-fun g (Int) Int)
   (declare-const a Int)
   (declare-const b Int)
   (declare-const c Int)
   (assert (forall ((x Int))
                   (! (= (f (g x)) x)
                      :pattern ((f (g x))))))
   (assert (= (g a) c))
   (assert (= (g b) c))
   (assert (not (= a b)))
   (check-sat)
   </pre>
    
   <p>When the more permissive pattern <tt>(g x)</tt> is used. Z3 proves the formula to be unsatisfiable.
   More restrive patterns minimize the number of instantiations (and potentially improve performance), but
   they may also make Z3 "less complete". 
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/2DWu?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (set-option :smt.auto-config false) ; disable automatic self configuration
   (set-option :smt.mbqi false) ; disable model-based quantifier instantiation
   (declare-fun f (Int) Int)
   (declare-fun g (Int) Int)
   (declare-const a Int)
   (declare-const b Int)
   (declare-const c Int)
   (assert (forall ((x Int))
                   (! (= (f (g x)) x)
                      :pattern ((g x)))))
   (assert (= (g a) c))
   (assert (= (g b) c))
   (assert (not (= a b)))
   (check-sat)
   </pre>
   
   <p>
   Some patterns may also create long instantiation chains.
   Consider the following assertion.
   </p>
   
   <pre class="listing">
   (assert (forall (x Type) (y Type) 
     (! (=&gt; (subtype x y) (subtype (array-of x) (array-of y)))
        :pattern ((subtype x y))
     ))
   </pre>
   
   <p>
   The axiom gets instantiated whenever there is some 
   ground term of the form <tt>(subtype s t)</tt>. 
   The instantiation causes a fresh ground term
   <tt>(subtype (array-of s) (array-of t))</tt>, which enables a new instantiation. 
   This undesirable situation is called a <b>matching loop</b>.
   Z3 uses many heuristics to break matching loops.
   </p>
   
   <p>
   Before elaborating on the subtleties, we should address an important first question.
   What defines the terms that are created during search?
   In the context of most SMT solvers, and of the Simplify theorem prover,
   terms exist as part of the input formula, they are of course also created 
   by instantiating quantifiers, but terms are also implicitly created when 
   equalities are asserted. 
   The last point means that terms are considered up to congruence and
   pattern matching takes place modulo ground equalities. We call the
   matching problem <b>E-matching</b>.
   For example, if we have the following equalities:
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/pO?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (set-option :smt.auto-config false) ; disable automatic self configuration
   (set-option :smt.mbqi false) ; disable model-based quantifier instantiation
   (declare-fun f (Int) Int)
   (declare-fun g (Int) Int)
   (declare-const a Int)
   (declare-const b Int)
   (declare-const c Int)
   (assert (forall ((x Int))
                   (! (= (f (g x)) x)
                      :pattern ((f (g x))))))
   (assert (= a (g b)))
   (assert (= b c))
   (assert (not (= (f a) c)))
   (check-sat)
   </pre>
   
   <p>
   The terms <tt>(f a)</tt> and <tt>(f (g b))</tt> are equal modulo the equalities.
   The pattern <tt>(f (g x))</tt> can be matched and <tt>x</tt> bound to <tt>b</tt>
   (and the equality <tt>(= (f (g b)) b)</tt> is deduced.
   </p>
   
   <p>
   While E-matching is an NP-complete problem, the main sources of overhead 
   in larger verification problems comes
   from matching thousands of patterns in the context of an evolving set
   of terms and equalities. Z3 integrates an efficient E-matching engine
   using term indexing techniques.
   </p>
   
   <h3>Multi-patterns</h3>
   
   <p>In some cases, there is no pattern that contains all bound variables and does not contain interpreted symbols.
   In these cases, we use multi-patterns.
   In the following example, the quantified formula states that <tt>f</tt> is injective.
   This quantified formula is annotated with the multi-pattern <tt>(f x) (f y)</tt>
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/5bH?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-sort A)
   (declare-sort B)
   (declare-fun f (A) B)
   (assert (forall ((x A) (y A))
                   (! (=&gt; (= (f x) (f y)) (= x y))
                      :pattern ((f x) (f y))
                      )))
   (declare-const a1 A)
   (declare-const a2 A)
   (declare-const b B)
   (assert (not (= a1 a2)))
   (assert (= (f a1) b))
   (assert (= (f a2) b))
   (check-sat)
   </pre>
   
   <p>
   The quantified formula  is instantiated for every pair of occurrences of <tt>f</tt>.
   A simple trick allows formulating injectivity of <tt>f</tt> in such a way that only a linear number
   of instantiations is required. The trick is to realize that <tt>f</tt> is injective if and only if
   it has a partial inverse. 
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/F?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (declare-sort A)
   (declare-sort B)
   (declare-fun f (A) B)
   (declare-fun f-inv (B) A)
   (assert (forall ((x A))
                   (! (= (f-inv (f x)) x)
                      :pattern ((f x))
                      )))
   (declare-const a1 A)
   (declare-const a2 A)
   (declare-const b B)
   (assert (not (= a1 a2)))
   (assert (= (f a1) b))
   (assert (= (f a2) b))
   (check-sat)
   </pre>
   
   <h3>No patterns</h3>
   
   <p>
   The annotation <tt>:no-pattern</tt> can be used to instrument Z3 
   not to use a certain sub-expression as a pattern.
   The pattern inference engine may otherwise choose arbitrary sub-expressions
   as patterns to direct quantifier instantiation. 
   </p>
   
   <h3>Model-based Quantifier Instantiation</h3>
   
   <p>
   The model-based quantifier instantiation (MBQI)  is essentially a
   counter-example based refinement loop, where candidate models are
   built and checked. When the model checking step fails, it creates new
   quantifier instantiations. The models are returned as simple
   functional programs. In the following example, the model provides an interpretation
   for function <tt>f</tt> and constants <tt>a</tt> and <tt>b</tt>. One can easily check that the
   returned model does indeed satisfy the quantifier.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/YS?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (set-option :smt.mbqi true)
   (declare-fun f (Int Int) Int)
   (declare-const a Int)
   (declare-const b Int)
   
   (assert (forall ((x Int)) (&gt;= (f x x) (+ x a))))
   
   (assert (&lt; (f a b) a))
   (assert (&gt; a 0))
   (check-sat)
   (get-model)
   
   (echo "evaluating (f (+ a 10) 20)...")
   (eval (f (+ a 10) 20))
   </pre>
   
   <p>
   The command <tt>eval</tt>
   evaluates an expression in the last model produced by Z3. It is essentially
   executing the "function program" produced by Z3.
   </p>
   
   <p>
   MBQI is a decision procedure for several useful fragments.
   It may find models even for formulas that are not in any of these fragments.
   We describe some of these fragments.
   </p>
   
   <h4>Effectively Propositional</h4>
   
   <p>
   The <b>effectively propositional</b> class of formulas (aka The Bernays-Schonfinkel class)
   is a decidable fragment of first-order logic formulas. It corresponds to formulas which, 
   when written in prenex normal form contain only constants, universal quantifiers, and
   functions that return boolean values (aka predicates).
   </p>
   <p>
   Problems arising from program verification often involve establishing facts of
   quantifier-free formulas, but the facts themselves use relations and functions that are conveniently axiomatized
   using a background theory that uses quantified formulas. One set of examples of this situation
   comprise of formulas involving partial-orders. The following example axiomatizes a <tt>subtype</tt>
   partial order relation that has the <b>tree property</b>. That is, if <tt>x</tt> and <tt>y</tt> are subtypes
   of <tt>z</tt>, then <tt>x</tt> is a subtype of <tt>y</tt> or <tt>y</tt> is a subtype of <tt>x</tt>.
   The option <tt>(set-option :model.compact true)</tt> instructs Z3 to eliminate trivial redundancies from the
   generated model. In this example, Z3 also creates a finite interpretation for the uninterpreted sort <tt>T</tt>.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/cF02?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   ; click on edit to see the example
   </pre>
   
   <p>
   Note that it uses two auxiliary functions (<tt>subtype!25</tt> and <tt>k!24</tt>) that were not
   part of your formula. They are auxiliary definitions created by Z3 during the model construction procedure.
   We can also ask "questions" by using the <tt>eval</tt> command. For example,
   </p>
   <pre class="listing">
   (eval (subtype int-type complex-type))
   </pre>
   <p>
   executes (evaluates) the given expression using the produced functional program (model).
   </p>
   <p>
   Constraints over sets (Boolean Algebras) can be encoded into this
   fragment by treating sets as unary predicates and lifting equalities
   between sets as formula equivalence. 
   <p/>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/YR?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   ; click on edit to see the example
   </pre>
   
   <h4>Stratified Sorts Fragment</h4>
   
   <p>The <b>statified sorts fragment</b> is another decidable fragment of
   many sorted first-order logic formulas. It corresponds to formulas which, 
   when written in prenex normal form, 
   there is a function <tt>level</tt> from sorts to naturals,
   and for every function </p>
   <pre class="listing">
   (declare-fun f (S_1 ... S_n) R)
   </pre>
   <p>
   <tt>level(R) &lt; level(S_i)</tt>. 
   </p>
   
   <h4>Array Property Fragment</h4>
   
   <p>The <b>array property fragment</b> can encode properties about unidimensional, and is
   strong enough to say an array is sorted. More information about this fragment can be
   found in the paper <a href="https://web.archive.org/web/20210125011020/http://academic.research.microsoft.com/Paper/1843442.aspx" target="_blank">What's Decidable About Arrays?</a>.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/H?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   ; click on edit to see the example
   </pre>
   
   <h4>List Fragment</h4>
   
   <p>The <b>list fragment</b> can encode properties about data-structures such as lists.
   For each quantified axiom <tt>q</tt> in this fragment, there is an "easy" way to satisfy <tt>q</tt>.
   More information about this fragment can be
   found in the paper <a href="https://web.archive.org/web/20210125011020/http://www.cs.berkeley.edu/~necula/Papers/verifier-cav05.pdf" target="_blank">
   Data Structure Specifications
   via Local Equality Axioms</a>.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/4Oplu?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   ; click on edit to see the example
   </pre>
   
   <h4>Essentially (Almost) Uninterpreted Fragment</h4>
   
   <p>The essentially/almost uninterpreted fragment subsumes the previous fragments, and uses
   a more relaxed notion of stratification. 
   More information about this fragment can be
   found in the paper <a href="https://web.archive.org/web/20210125011020/http://research.microsoft.com/en-us/um/people/leonardo/ci.pdf" target="_blank">
   Complete instantiation for quantified formulas in
   Satisfiabiliby Modulo Theories.
   </a> The model based quantifier instantiation approach used in Z3 is also described in this paper.
   Stratified data-structures (such as arrays of pointers) can be encoded in this fragment. </p>
    
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/W?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   ; click on edit to see the example.
   </pre>
   
   <p>
   Shifts on streams (or arrays) can also be encoded.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/5Lb?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (set-option :smt.mbqi true)
   ;; f an g are "streams"
   (declare-fun f (Int) Int)
   (declare-fun g (Int) Int)
   
   ;; the segment [a, n + a] of stream f is equal
   ;; to the segment [0, n] of stream g.
   (declare-const n Int)
   (declare-const a Int)
   (assert (forall ((x Int)) (=&gt; (and (&lt;= 0 x) (&lt;= x n))
                                 (= (f (+ x a)) (g x)))))
   
   ;; adding some constraints to a
   (assert (&gt; a 10))
   (assert (&gt;= (f a) 2))
   (assert (&lt;= (g 3) (- 10)))
   
   (check-sat)
   (get-model)
   </pre>
   
   <h4>Quantified Bit-Vector Formulas</h4>
   
   <p>
   A <b>quantified bit-Vector formula</b> (QBVF) is a many sorted
   first-order logic formula where the sort of every variable is a
   bit-vector sort.  The QBVF satisfiability problem, is the problem of
   deciding whether a QBVF is satisfiable modulo the theory of
   bit-vectors. This problem is decidable because every universal
   (existental) quantifier can be expanded into a conjunction
   (disjunction) of potentially exponential, but finite size. 
   A distinguishing feature in QBVF is the support for uninterpreted
   function and predicate symbols. More information about this fragment
   can be found in the paper 
   <a href="https://web.archive.org/web/20210125011020/http://research.microsoft.com/en-us/um/people/leonardo/fmcad10.pdf" target="_blank">
   Efficiently Solving Quantified Bit-Vector Formulas</a>.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/4TZ7?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (set-option :smt.mbqi true)
   (define-sort Char () (_ BitVec 8))
   
   (declare-fun f  (Char) Char)
   (declare-fun f1 (Char) Char)
   (declare-const a Char)
   
   (assert (bvugt a #x00))
   (assert (= (f1 (bvadd a #x01)) #x00))
   (assert (forall ((x Char)) (or (= x (bvadd a #x01)) (= (f1 x) (f x)))))
   
   (check-sat)
   (get-model)
   </pre>
   
   <h4>Conditional (and Pseudo) Macros</h4>
   
   <p> Quantifiers defining "macros" are also automatically detected by the Model Finder.
   In the following example, the first three quantifiers are defining <tt>f</tt> by cases.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/G5?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   (set-option :smt.mbqi true)
   (declare-fun f (Int) Int)
   (declare-fun p (Int) Bool)
   (declare-fun p2 (Int) Bool)
   (declare-const a Int)
   (declare-const b Int)
   (declare-const c Int)
   (assert (forall ((x Int)) 
                   (=&gt; (not (p x)) (= (f x) (+ x 1)))))
   (assert (forall ((x Int)) 
                   (=&gt; (and (p x) (not (p2 x))) (= (f x) x))))
   (assert (forall ((x Int)) 
                   (=&gt; (p2 x) (= (f x) (- x 1)))))
   (assert (p b))
   (assert (p c))
   (assert (p2 a))
   (assert (&gt; (f a) b))
   (check-sat)
   (get-model)
   </pre>
   
   <h4>My formula is not in any of the fragments above</h4>
   
   <p>
   Even if your formula is not in any of the fragments above.
   Z3 may still find a model for it. For example, 
   The following simple example is not in the fragments described above.
   </p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/v?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   ; click on edit to see the example.
   </pre>
   
   <p>
   The Z3 preprocessor has many options that may improve the performace of the model
   finder. In the following example, <tt>:macro-finder</tt> will expand quantifiers representing macros
   at preprocessing time.</p> 
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/yl?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   ; click on edit to see the example.
   </pre>
   
   <p>It is very effective in this benchmark since it contains many
   quantifiers of the form
   </p>
   <pre class="listing">
   forall x.  p(x) = ....
   </pre>
   
   <p>
   The Z3 model finder is more effective if the input formula does not contain nested quantifiers.
   If that is not the case for your formula, you can use the option</p>
   <pre class="listing">
   (set-option :smt.pull-nested-quantifiers true)
   </pre>
   <p> 
   The following challenge problem from the paper <a href="https://web.archive.org/web/20210125011020/http://academic.research.microsoft.com/Paper/615910.aspx" target="_blank">
   SEM: a system for enumerating models</a> is proved to be unsatisfiable in less than one second by Z3.</p>
   
   <a class="listinglink" target="default" href="/web/20210125011020/https://www.rise4fun.com/Z3/WH?frame=1&amp;menu=0&amp;course=1">load in editor</a><pre class="listing">
   ; click on edit to see the example.
   </pre>
   
   <h2>Conclusion</h2>
   
   <p>
   Z3 is an efficient theorem prover used in many software testing, analysis and verification applications.
   In this tutorial, we covered its main capabilities using the textual interface.
   However, most applications use the Z3 programmatic 
   <a href="https://web.archive.org/web/20210125011020/http://research.microsoft.com/en-us/um/redmond/projects/z3/documentation.html" target="_blank">API</a>
   to access these features.
   </p>
   