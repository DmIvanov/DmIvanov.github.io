---
id: 167
title: Code optimization for Swift and Objective-C
date: 2018-12-02T11:46:12-02:00
author: topolog
layout: post
guid: https://dmtopolog.com/?p=167
permalink: /code-optimization-for-swift-and-objective-c/
image: /wp-content/uploads/2018/12/clang_pipeline.png
categories:
  - Tech Blog
tags:
  - build process
  - compiler
  - iOS
---
<p class="p1">
  <span class="s1">Some time ago I needed to switch off asserts for one of the configurations in our project. I kinda read somewhere that whether or not asserts compile to the resulting binary depends primarily on the code optimisation level. But I wondered if there was any way to control the asserts without changing the optimisation level? What if I don&#8217;t want to touch it for the asserts only? Do we have separate optimisation levels for Swift and Objective-C code? What do they mean, what are they influence on? How do two compilers in the mixed Swift/ObjC project work together optimising the code? What specifically the compiler does for each optimisation level? </span>
</p>

<p class="p1">
  <span class="s1">To answer these question I tried to get some details about xCode build pipeline, about how the compilers work and where is the place for the code optimization in the process. Tell you in advance that I couldn&#8217;t find satisfying answers for all of my questions, but the picture of the building process, compilers and optimisation levels became more clear for me.</span>
</p>

&nbsp;

### <span class="s1">Compilers in building iOS apps</span> {.p1}

> <p class="p1">
>   <span class="s1">The compiler is just a big function which takes a string and returns a big int (Scott Owens ICFP 2017)</span>
> </p>

<p class="p1">
  <span class="s1">First some general theory. There are several ways to build an iOS/macOS project: using xCode, xcodebuild from the command line, fastlane (which uses xcodebuild under the hood), manual scripts or something else. Anyway you need to do the same set of actions to make the app package which can be installed on a device. </span>
</p>

<pre class="p1">$ swiftc -module-name YourModule -target arm64-apple-ios12.0 -swift-version 4.2 ...
$ clang -x objective-c -arch arm64 ... YourFile.m -o YourFile.o
$ ld -o YourProject -framework YourFile.o ...
$ actool --app-icon AppIcon ... Assets.xcassets
$ ...</pre>

<p class="p1">
  <span class="s1">or</span>
</p>

<pre class="lang:swift decode:true">$ xcodebuild -target YourProject -xcconfig your_configuration_file.xcconfig</pre>

<p class="p1">
  <span class="s1">Code compilation is one of the steps you need to do (together with linking, bundling the resources, code signing and so on).</span>
</p>

<p class="p1">
  <span class="s1">Originally we have our source code written in some high level programming language: ObjC, Swift, C++ (you also have some resources like images, strings, fonts etc., but they are out of scope of this topic). Compiler&#8217;s job is to translate our high level source code into low level machine code to create an executable program. There are different types of compilers but in our iOS/macOS world we deal with ones which can be divided into a frontend and a backend. The frontend takes our source code and converts it into some intermediate language understandable for the backend. The backend takes this intermediate representation and turns it into machine code with the regard of the specific operation system and processor architecture. That&#8217;s the essence of this two entities.</span>
</p>

<p class="p1">
  <span class="s1">There are different pipelines for Swift and ObjC but both of them are backed by LLVM. </span>
</p>

<p class="p1">
  <span class="s1">LLVM is a big project including LLVM Core as a backend, Clang as &#8220;LLVM native&#8221; frontend for C language family (C/ObjC/C++/ObjC++) and interaction protocol between the frontend and the backend. The major part of this protocol is LLVM IR (Intermediate Representation) &#8211; the intermediate language understandable for both the front and the back. It doesn&#8217;t matter for LLVM Core exactly what frontend has produced the IR-code out of the source code. The Core just takes this representation works with it and generates an executable or dynamic library.</span>
</p>

<p class="p1">
  <span class="s1">Although Clang is sometimes considered as a part of LLVM, the entire idea of the LLVM project is that the frontend is decoupled from the backend. Swift development team (which includes several members of LLVM community) managed to leverage it and use the same LLVM Core as a backend for the swift compiler.</span>
</p>

&nbsp;

### <span class="s1">Objective-C pipeline</span> {.p1}

<p class="p1">
  <span class="s1">So for good old ObjC code the pipeline looks like this:</span>
</p>

<p class="p1">
  <span class="s1"><img class="wp-image-168 aligncenter" src="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/clang_pipeline.png?resize=688%2C101&#038;ssl=1" alt="" width="688" height="101" srcset="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/clang_pipeline.png?resize=300%2C44&ssl=1 300w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/clang_pipeline.png?resize=768%2C113&ssl=1 768w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/clang_pipeline.png?resize=1024%2C151&ssl=1 1024w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/clang_pipeline.png?resize=1600%2C236&ssl=1 1600w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/clang_pipeline.png?w=1376&ssl=1 1376w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/clang_pipeline.png?w=2064&ssl=1 2064w" sizes="(max-width: 688px) 100vw, 688px" data-recalc-dims="1" /></span>
</p>

<p class="p1">
  <span class="s1">You can see that the first conversion Clang makes over your code is creation of AST&#8217; (Abstract Syntax Tree) &#8211; the representation where all the functions, operators, variables, declarations.. are nodes of the huge semantic tree. Here is an example of AST:</span>
</p>

<pre class="lang:swift decode:true">$ cat test.cc
 int f(int x) {
 &nbsp; int result = (x / 42);
 &nbsp; return result;
 }</pre>

<p class="p1">
  <span class="s1">turns into</span>
</p>

<pre class="lang:swift decode:true">$ clang -Xclang -ast-dump -fsyntax-only test.cc
 TranslationUnitDecl 0x5aea0d0 &lt;&lt;invalid sloc&gt;&gt;
 ... cutting out internal declarations of clang ...
 `-FunctionDecl 0x5aeab50 &lt;test.cc:1:1, line:4:1&gt; f 'int (int)'
 &nbsp; |-ParmVarDecl 0x5aeaa90 &lt;line:1:7, col:11&gt; x 'int'
 &nbsp; `-CompoundStmt 0x5aead88 &lt;col:14, line:4:1&gt;
 &nbsp; &nbsp; |-DeclStmt 0x5aead10 &lt;line:2:3, col:24&gt;
 &nbsp; &nbsp; | `-VarDecl 0x5aeac10 &lt;col:3, col:23&gt; result 'int'
 &nbsp; &nbsp; | &nbsp; `-ParenExpr 0x5aeacf0 &lt;col:16, col:23&gt; 'int'
 &nbsp; &nbsp; | &nbsp; &nbsp; `-BinaryOperator 0x5aeacc8 &lt;col:17, col:21&gt; 'int' '/'
 &nbsp; &nbsp; | &nbsp; &nbsp; &nbsp; |-ImplicitCastExpr 0x5aeacb0 &lt;col:17&gt; 'int' &lt;LValueToRValue&gt;
 &nbsp; &nbsp; | &nbsp; &nbsp; &nbsp; | `-DeclRefExpr 0x5aeac68 &lt;col:17&gt; 'int' lvalue ParmVar 0x5aeaa90 'x' 'int'
 &nbsp; &nbsp; | &nbsp; &nbsp; &nbsp; `-IntegerLiteral 0x5aeac90 &lt;col:21&gt; 'int' 42
 &nbsp; &nbsp; `-ReturnStmt 0x5aead68 &lt;line:3:3, col:10&gt;
 &nbsp; &nbsp; &nbsp; `-ImplicitCastExpr 0x5aead50 &lt;col:10&gt; 'int' &lt;LValueToRValue&gt;
 &nbsp; &nbsp; &nbsp; &nbsp; `-DeclRefExpr 0x5aead28 &lt;col:10&gt; 'int' lvalue Var 0x5aeac10 'result' 'int'</pre>

<p class="p1">
  <span class="s1">Then AST is being converted into LLVM IR which is more low level (less human readable) but some part&#8217;s of the source code still can be understood (<a href="http://llvm.org/docs/LangRef.html">here</a> is a detailed description of the language):</span>
</p>

<pre class="lang:swift decode:true">int main()
{
&nbsp; return 0;
}</pre>

<p class="p1">
  <span class="s1"> turns into</span>
</p>

<pre class="lang:swift decode:true">define i32 @main() #0 {
 &nbsp; %1 = alloca i32, align 4
 &nbsp; store i32 0, i32* %1
 &nbsp; ret i32 0
 }</pre>

<p class="p1">
  <span class="s1">LLVM IR is being passed from Clang to LLVM Core where the code is optimized (if applicable) and converted to a target specific machine code. As a result we have a bunch of object files (*.o) which are linked together later on and merged into an executable or dynamic library. The output of this last stage is typically called an “a.out”, “.dylib” or “.so” file. </span>
</p>

<p class="p1">
  <span class="s1">As you could already see LLVM Core is the place where the code is optimized, and Intermediate Representation is specifically the source of these optimizations.</span>
</p>

<p class="p1">
  <span class="s1">Here are <strong>LLVM optimization levels</strong> with brief corresponding descriptions:</span>
</p>

<li class="p1">
  <span class="s1"><strong>None [-O0]</strong>&nbsp;</span>The compiler does not attempt to optimize code. With this setting, the compiler’s goal is to reduce the cost of compilation and to make debugging produce the expected results. Statements are independent: if you stop the program with a breakpoint between statements, you can then assign a new value to any variable or change the program counter to any other statement in the function and get exactly the results you would expect from the source code. Use this option during development when you are focused on solving logic errors and need a fast compile time. Do not use this option for shipping your executable.
</li>
<li class="p1">
  <strong><span class="s1">Fast [-O, O1]&nbsp;</span></strong>The compiler performs simple optimizations to boost code performance while minimizing the impact to compile time. This option also uses more memory during compilation.
</li>
<li class="p1">
  <strong><span class="s1">Faster [-O2]&nbsp;</span></strong>The compiler performs nearly all supported optimizations that do not require a space-time tradeoff. The compiler does not perform loop unrolling or function inlining with this option. This option increases both compilation time and the performance of generated code.
</li>
<li class="p1">
  <strong><span class="s1">Fastest [-O3]&nbsp;</span></strong>The compiler performs all optimizations in an attempt to improve the speed of the generated code. This option can increase the size of generated code as the compiler performs aggressive inlining of functions. (This option is generally not recommended.)
</li>
<li class="p1">
  <strong><span class="s1">Fastest, Smallest [-Os]&nbsp;</span></strong>The compiler performs all optimizations that do not typically increase code size. This is the preferred option for shipping code because it gives your executable a smaller memory footprint.
</li>
<li class="p1">
  <strong><span class="s1">Fastest, Aggressive optimization [-Ofast]&nbsp;</span></strong>This setting enables ‘Fastest’ but also enables aggressive optimizations that may break strict standards compliance but should work well on well-behaved code.
</li>

<p class="p1">
  <span class="s1">Depending on the level LLVM makes several passes through given IR analysing and changing the code. Looking for the lists of specific passes for each optimization level I didn&#8217;t manage to find anything other that <a href="https://stackoverflow.com/a/15548189/8915388">this SO answer</a> which arises more additional questions. The official list of LLVM passes (without separation to different levels) can be found <a href="http://llvm.org/docs/Passes.html">here</a>.</span>
</p>

<p class="p1">
  <span class="s1">When creating a new project xCode sets up 2 configurations with following default values for the LLVM optimization levels: </span>
</p>

<li class="p1">
  <span class="s1"> Debug:<span class="Apple-converted-space">&nbsp; </span><strong>-O0 (none)</strong> &#8211; the fastest compile time, the easiest debugging</span>
</li>
<li class="p1">
  <span class="s1"> Release:<strong> -Os (fastest, smallest)</strong> &#8211; the best combination of small binary size and fast runtime execution</span>
</li>

<p class="p1">
  <span class="s1">Interestingly enough, configuration option responsible for C-family language optimisation calls (in project.pbxproj file) is called GCC_OPTIMIZATION_LEVEL. If you dive a bit in the history of xCode you will see that till xCode 3 the GCC (<a href="https://gcc.gnu.org/">GNU Compiler Collection</a>) was the compiler used by the IDE, then there was a transition period when the user could change the compiler in the target settings and with the release of xCode 5 CLang became the only option (<a href="https://useyourloaf.com/blog/compiler-options-in-xcode-gcc-or-llvm">a piece of such history</a>). Seems like the target property name still has that legacy.</span>
</p>

&nbsp;

### <span class="s1">Swift pipeline</span> {.p1}

<p class="p1">
  <span class="s1">Swift is young and rapidly developing language. So as it&#8217;s compiler. In terms of code optimisation some features are being changed and new features appear almost every year. (What I&#8217;m writing here about is valid for Swift 4.2 and XCode 10)</span>
</p>

<p class="p1">
  <span class="s1">Let&#8217;s take a look at the compilation pipeline for Swift:</span>
</p>

<p class="p1">
  <img class="aligncenter wp-image-169" src="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/swiftc_pipeline.png?resize=688%2C73&#038;ssl=1" alt="" width="688" height="73" srcset="https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/swiftc_pipeline.png?resize=300%2C32&ssl=1 300w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/swiftc_pipeline.png?resize=768%2C81&ssl=1 768w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/swiftc_pipeline.png?resize=1024%2C109&ssl=1 1024w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/swiftc_pipeline.png?resize=1600%2C170&ssl=1 1600w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/swiftc_pipeline.png?w=1376&ssl=1 1376w, https://i1.wp.com/dmtopolog.com/wp-content/uploads/2018/12/swiftc_pipeline.png?w=2064&ssl=1 2064w" sizes="(max-width: 688px) 100vw, 688px" data-recalc-dims="1" />
</p>

<p class="p1">
  <span class="s1">You can see lots of similarities with the one for ObjC: not only the same backend but also the frontend essentially works similarly. Lexical analysis, tokenization (separating some lexical items from the raw string), building AST, type checking. The main difference is the presence of SIL (Swift Intermediate Language) &#8211; another intermediate code representation between AST and LLVM IR.</span>
</p>

<p class="p1">
  <span class="s1">The Swift development team took Clang as an example for a frontend, tried to use all Clang&#8217;s advantages and compensate some flaws. One of that flaws was inability to implement some high level analysis, reliable diagnostics and optimizations for which neither ATS nor LLVM IR were a proper material. So SIL was the solution for this problem.</span>
</p>

<p class="p1">
  <span class="s1">Here is an example of how swift function looks in SIL:</span>
</p>

<pre class="lang:swift decode:true ">func fibonacci(lim: Int) {
    var a = 0, b = 1
    while b &lt; lim {
        print(b)
        (a, b) = (b, a + b)
    }
}</pre>

turns into

<pre class="">sil @fibonacci: $(Swift.Int) -&gt; () {
  entry(%limi: $Swift.Int):
    %lim = struct_extract %limi: $Swift.Int, #Int.value
    %print = function_ref @print: $(Swift.Int) -&gt; ()
    %a0 = integer_literal $Builtin.Int64, 0
    %b0 = integer_literal $Builtin.Int64, 1
    %lt = builtin "icmp_lt_Int64"(%b: $Builtin.Int64, %lim: $Builtin.Int64): $Builtin.Int1
    cond_br %lt: $Builtin.Int1, body, exit
    br loop(%a0: $Builtin.Int64, %b0: $Builtin.Int64)
  loop(%a: $Builtin.Int64, %b: $Builtin.Int64):
  body:
    %b1 = struct $Swift.Int (%b: $Builtin.Int64)
    apply %print(%b1) : $(Swift.Int) -&gt; ()
    %c = builtin "add_Int64"(%a: $Builtin.Int64, %b: $Builtin.Int64): $Builtin.Int64
    br loop(%b: $Builtin.Int64, %c: $Builtin.Int64)
  exit:
    %unit = tuple ()
    return %unit: $()
}</pre>

<p class="p1">
  <span class="s1">So.. what&#8217;s about optimizations? You might already got it that for swift code we have the same low level LLVM optimizations made over LLVM IR, plus we have some set of additional optimizations made on a higher level by swift compiler over SIL. </span>
</p>

<p class="p1">
  <span class="s1">At different steps the parts of the Swift compiler produce different kinds of SIL. At the beginning SILGen produces so called <em>raw SIL</em>. Raw SIL is just another representation of your code: no diagnostics, no changes, potential data flow errors. A small set of optimisations is being made on the raw SIL (even if you have all the optimizations switched off) &#8211; <strong>Guaranteed Optimization Passes and Diagnostic Passes</strong>:</span>
</p>

<li class="p1">
  <span class="s1"><strong>Mandatory inlining</strong> inlines calls to &#8220;transparent&#8221; functions.</span>
</li>
<li class="p1">
  <span class="s1"><strong>Memory promotion</strong> is implemented as two optimization phases, the first of which performs capture analysis to promote alloc_box instructions to alloc_stack, and the second of which promotes non-address-exposed alloc_stack instructions to SSA registers.</span>
</li>
<li class="p1">
  <span class="s1"><strong>Constant propagation</strong> folds constant expressions and propagates the constant values. If an arithmetic overflow occurs during the constant expression computation, a diagnostic is issued.</span>
</li>
<li class="p1">
  <span class="s1"><strong>Return analysis</strong> verifies that each function returns a value on every code path and doesn&#8217;t &#8220;fall off the end&#8221; of its definition, which is an error. It also issues an error when a noreturn function returns.</span>
</li>
<li class="p1">
  <span class="s1"><strong>Critical edge splitting</strong> splits all critical edges from terminators that don&#8217;t support arbitrary basic block arguments (all non cond_branch terminators).</span>
</li>

<p class="p1">
  <span class="s1">If all diagnostic passes succeed, the final result is the <em>canonical SIL</em> for the program. Canonical SIL means that<span class="Apple-converted-space">&nbsp; </span>guaranteed optimizations and diagnostics were done, so all the data flow errors should be eliminated and certain instructions must be canonicalized to simpler forms. And here is the turn for <strong>General Optimization Passes</strong>, which are done (or not done) according to the optimization level you set up:</span>
</p>

<li class="p1">
  <span class="s1"><strong>Generic Specialization</strong> analyses specialized calls to generic functions and generates new specialized version of the functions. Then it rewrites all specialized usages of the generic to a direct call of the appropriate specialized function.</span>
</li>
<li class="p1">
  <span class="s1"><strong>Witness and VTable Devirtualization</strong> for a given type looks up the associated method from a class&#8217;s vtable or a type witness table and replaces the indirect virtual call with a call to the mapped function.</span>
</li>
<li class="p1">
  <strong><span class="s1">Performance Inlining</span></strong>
</li>
<li class="p1">
  <strong><span class="s1">Reference Counting Optimizations</span></strong>
</li>
<li class="p1">
  <strong><span class="s1">Memory Promotion/Optimizations</span></strong>
</li>
<li class="p1">
  <span class="s1"><strong>High-level domain specific optimizations.</strong> The Swift compiler implements high-level optimizations on basic Swift containers such as Array or String. Domain specific optimizations require a defined interface between the standard library and the optimizer.</span>
</li>

<p class="p1">
  <span class="s1">Until Swift 4.1 swift compiler had only two optimization levels: [-Onone] and [-O]. But now there are already 3 different levels:</span>
</p>

<li class="p1">
  <strong><span class="s1">None [-Onone]</span></strong>
</li>
<li class="p1">
  <strong><span class="s1">Optimize for speed [-O]</span></strong>
</li>
<li class="p1">
  <strong><span class="s1">Optimize for size [-Osize]</span></strong>
</li>

<p class="p1">
  <span class="s1">Optimization for speed includes all the General Optimization Passes listed before. Optimization for size (appeared in Swift 4.1) is claimed to have the same optimizations but with the additional parameter &#8211; size limitation. Basically the compiler tries to reduce code duplication when optimizing the code. Specifically it tries to be a bit smarter in function inlining (not doing the job if it doesn&#8217;t decrease a code size). Also the compiler is able to extract some code into a helper functions which also decrease the resulting code size. Optimization for size is an analog of <strong>Fastest, Smallest</strong> optimization made by LLVM.</span>
</p>

<p class="p1">
  <span class="s1">The trade-off between the two optimization level is as following: size optimization can save you 5%-30% of the code size, performance optimization can improve the speed of the code up to 5%.</span>
</p>

<p class="p1">
  <span class="s1">Of course we also shouldn&#8217;t forget about the Whole-Module Optimization (appeared in Swift 3). Now this property sits in the project settings as Swift compilation mode and has two options:</span>
</p>

<li class="p1">
  <span class="s1">Incremental (Single file in Xcode 9.3)</span>
</li>
<li class="p1">
  <span class="s1">Whole module</span>
</li>

<p class="p1">
  <span class="s1">The essence of the optimisation is intelligibly described in the official <a href="https://swift.org/blog/whole-module-optimizations">swift blog.</a> But briefly it&#8217;s not related to how the compiler process the source code, but what exactly it takes as an input for its work. In incremental (single file) mode the compiler works separately with each swift file doing all the analysis and optimisations. In a whole-module mode it &#8220;combines&#8221; all the swift source files in one module all together and takes the entire module as a source for all the processing. Of course every mode has its trade-offs. Whole-module optimisation has several significant benefits based on the bigger context for the optimizations (whole module instead of just one file) &#8211; when the compiler knows more about the code it can do much more for you in terms of optimisation. But it&#8217;s clear that in this case the compiler needs to recompile the entire module every time you run it, even if you made just a small change in one file (as there are know separate files for the compiler). That&#8217;s why by default in the new xCode project the debug configuration has an incremental compilation mode (almost no optimizations, the fastest incremental compilation) and the release configuration has whole-module optimisation switched on (no need in benefits of incremental compilation, better optimisations).</span>
</p>

<p class="p1">
  <span class="s1">There is also kind of an optimization option <strong>-Ounchecked</strong> compiler flag which was one of the options of the Optimization level in previous xCode versions and now is called Disable Safety Checks. It&#8217;s responsible for removing some optimization passes related to things like int overflow. But there are <a href="https://forums.swift.org/t/deprecating-ounchecked/6928/10">some arguments</a> if it&#8217;s an &#8220;optimization level&#8221; in a first place, and where is it&#8217;s place in the compiler.</span>
</p>

<p class="p1">
  <span class="s1">We&#8217;ve already mentioned that there are 2 sets of optimisation passes for swift code: frontend optimizations (made by SIL optimizer) and backend optimization (made by LLVM Core). So you might think: &#8220;Wait.. if we have whole-module mode turned on and the compiler treats the entire module &#8211; which often means <em>the entire project</em> &#8211; as the big chunk of code, so we miss all the concurrency benefits. We cannot process different parts of the module simultaneously anymore&#8221;. And partly you are right. For the frontend the compilation of the entire module is indeed one single process. But it seems that Apple engineers managed to split the module for LLVM, so at least the backend can apply some concurrency to speed up the compilation.</span>
</p>

&nbsp;

### <span class="s1">How can we use this information in our daily routine?</span> {.p1}

<p class="p1">
  <span class="s1">Ok. The last question is why should we care about the optimization level? Why just not trust apple with its predefined project settings and don&#8217;t waste the limited space in your head with this information?</span>
</p>

<p class="p1">
  <span class="s1">Basically it&#8217;s that kind of knowledge which not just interesting to get, but also can suddenly be helpful in a future debugging in some cases which I cannot imagine now. But right here I can list at least couple of reasons why this can be useful.</span>
</p>

##### <span class="s1">Size/performance/time</span> {.p1}

<p class="p1">
  <span class="s1">There are several classical triangles like this when you have to prioritise the most important vertices because it&#8217;s all about trade-offs, if you pick one characteristic the other will inevitably suffer. Here we have size of the binary output, performance of the resulting code and time of compilation. So if we wan&#8217;t to achieve the smallest compilation time we should switch all the optimizations off and so the performance and the size will be worse than they could have been. If performance is our primary goal we should be ready to spare some extra time for the compiler and be ready that the resulting size might be a bit bigger because some performance optimizations cause code duplication. If the size is paramount so we will have not the best performance and not the minimal compilation time. </span>
</p>

<p class="p1">
  <span class="s1">Fortunately engineers working on the compiler understand that usually we can sacrifice some performance in sake of the app size and some size in sake of performance, so we have optimization mode with the best combination of this two factors (needless to say that compilation time is the price we pay for that). As a result we have <strong>[-Os]</strong> optimization in LLVM and <strong>[-Osize]</strong> in Swift compiler, which aim to give us the best traid-off for production.</span>
</p>

##### <span class="s1">Debugging</span> {.p1}

<p class="p1">
  <span class="s1">I&#8217;ve already written that we don&#8217;t usually need optimizations switched on for the debug configuration. Firstly, we want to decrease the compilation time as we run the compiler dozens times a day. But other than that we don&#8217;t want the compiler to rearrange our code much because in this case it is hard to debug it. If the resulting code running on the device or simulator is highly optimized, it might be impossible for the debugger to fire your breakpoints, step into the code line-by-line or expose some variables for you. Debugging is kind of a reverse engineering that the debugger makes for you to connect some piece of running compiled code with the source code you see in IDE, and optimisations make this job way more difficult for the debugger.</span>
</p>

<p class="p1">
  <span class="s1">Quite often there are several configurations in big production projects (more than just Debug and Release), and they might have different optimization levels. That&#8217;s definitely something you have to consider when switching from one configuration to another for debugging.</span>
</p>

##### <span class="s1">Assertions</span> {.p1}

<p class="p1">
  <span class="s1">Coming back to the original question about the asserts: first of all you need to understand that Swift and ObjC asserts are two different matters. So to handle them you have to separately communicate to both compilers.</span>
</p>

<p class="p1">
  <strong><span class="s1">Asserts in Swift</span></strong>
</p>

<p class="p1">
  <span class="s1">In case of Swift there are several types of asserts (you can read about them for instance <a href="https://blog.krzyzanowskim.com/2015/03/09/swift-asserts-the-missing-manual/">here</a>) by &#8220;assert&#8221; in the context of Swift I consider regular <code>assert(_:_:file:line:)</code> function</span>
</p>

<p class="p1">
  <span class="s1">Removing asserts is one of the optimizations that swift compiler does (or does not depending on your settings). So as it said in the <a href="https://developer.apple.com/documentation/swift/1541112-assert">documentation</a> if optimization level is -Onone you will have the asserts in your resulting code. If the level is different they will be optimized out of the code.</span>
</p>

<p class="p1">
  <span class="s1">But Swift compiler also has a specific flag -assert-config which overwrites the optimization level for the assertions. So if you have this flag set with some value it will cancel whatever the optimization level defines for the assertions. The flag has string values: Debug, Release, Unchecked, DisableReplacement. Debug and DisableReplacement values cause your swift asserts to remain in the resulting code; Release and Unchecked force the compiler to optimize them out. There is no dedicated property for it in the xCode project settings so you have to add it in the Other Swift Flags section (`-assert-config Unchecked` for instance).</span>
</p>

<p class="p1">
  <strong><span class="s1">Asserts in Objective-C</span></strong>
</p>

<p class="p1">
  <span class="s1">In ObjC there are two types of assertions: <code>assert(condition)</code> coming from C and <code>NSAssert(condition, comment)</code> &#8211; so called Foundation Assertions. </span>
</p>

<p class="p1">
  <span class="s1">The first hardcore one will be in the resulting code almost in all conditions regardless you configuration, optimisation level and the rest of the xCode build settings. The only way to switch it off that I&#8217;m aware of is to define NDEBUG=1 Preprocessor Macros (together with DEBUG=1 which usually defined by default). </span>
</p>

<p class="p1">
  <span class="s1">NSAssert is not so tough and handling it is easier: there is a bool property in the xCode build settings Enable Foundation Assertions (ENABLE_NS_ASSERTIONS). Also assertions are disabled if preprocessor macro NS_BLOCK_ASSERTIONS is defined. But it also has nothing to do with the optimization level.</span>
</p>

<p class="p1">
  <span class="s1">So eventually you don&#8217;t need to change any of the optimization levels in your project settings to switch the assertions on/off. Both compilers have special compilation flags for handling them.</span>
</p>

&nbsp;

### <span class="s1">Conclusion</span> {.p1}

<p class="p1">
  <span class="s1">There was my small journey started from the question about asserts and led me right into the compiler theory. On a high level I described the process of building the application from the source code, pointed out the place that code optimisation takes in this process. We also tried to figure out which types of optimizations are there and what exactly the compiler does to optimize your code. Eventually we got to know that optimization level has not so much to do optimizing out the asserts from the source code.</span>
</p>

&nbsp;

### _<span class="s1">Additional references:</span>_ {.p1}

<p class="p1">
  <strong><em><span class="s1">Clang:</span></em></strong>
</p>

<p class="p1">
  <a href="https://clang.llvm.org/docs/CommandGuide/clang.html#description"><em><span class="s1"> https://clang.llvm.org/docs/CommandGuide/clang.html#description</span></em></a><br /> <a href="https://clang.llvm.org/docs/CommandGuide/clang.html#code-generation-options"><em><span class="s1"> https://clang.llvm.org/docs/CommandGuide/clang.html#code-generation-options</span></em></a>
</p>

<p class="p1">
  <strong><em><span class="s1">LLVM:</span></em></strong>
</p>

<p class="p1">
  <em><span class="s1">Video: <a href="https://youtu.be/a5-WaD8VV38">A Brief Introduction to LLVM</a></span></em>
</p>

<p class="p1">
  <em><span class="s1">Video: <a href="https://youtu.be/b_T-eCToX1I">D. Dunbar “A New Architecture for Building Software”</a></span></em>
</p>

<p class="p1">
  <strong><em><span class="s1">LLVM optimization:</span></em></strong>
</p>

<p class="p1">
  <em><span class="s1"><a href="https://developer.apple.com/library/archive/documentation/General/Conceptual/MOSXAppProgrammingGuide/Performance/Performance.html"> https://developer.apple.com/library/archive/documentation/General/Conceptual/MOSXAppProgrammingGuide/Performance/Performance.html</a> (Compiler-Level Optimizations section)</span></em>
</p>

<p class="p1">
  <em><span class="s1"><a href="https://pewpewthespells.com/blog/buildsettings.html#gcc_optimization_level"> https://pewpewthespells.com/blog/buildsettings.html#gcc_optimization_level</a> (Xcode Build Settings Reference)</span></em>
</p>

<p class="p1">
  <strong><em><span class="s1">Swift compiler:</span></em></strong>
</p>

<p class="p1">
  <a href="https://modocache.io/reading-and-understanding-the-swift-driver-source-code"><em><span class="s1"> https://modocache.io/reading-and-understanding-the-swift-driver-source-code</span></em></a>
</p>

<p class="p1">
  <strong><em><span class="s1">Swift optimizations:</span></em></strong>
</p>

<p class="p1">
  <a href="https://swift.org/blog/osize/"><em><span class="s1"> https://swift.org/blog/osize/</span></em></a>
</p>

<p class="p1">
  <a href="https://swift.org/blog/whole-module-optimizations/"><em><span class="s1"> https://swift.org/blog/whole-module-optimizations/</span></em></a>
</p>

<p class="p1">
  <strong><em><span class="s1">SIL:</span></em></strong>
</p>

<p class="p1">
  <a href="https://github.com/apple/swift/blob/master/docs/SIL.rst"><em><span class="s1"> Documentation</span></em></a>
</p>

<p class="p1">
  <em><span class="s1">Video: <a href="https://youtu.be/Ntj8ab-5cvE">Joseph Groff & Chris Lattner Swift&#8217;s High-Level IR</a></span></em>
</p>