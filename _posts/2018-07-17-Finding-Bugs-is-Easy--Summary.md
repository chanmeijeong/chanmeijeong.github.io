---

title : "Finding Bugs is Easy"
date : 2018-07-17
category: Secure Coding

---

# THIS POST IS,
a summary of a paper called, 'Finding Bugs is Easy.'

**Finding Bugs is Easy**
**-by David Hovemeyer and William Pugh**

You can see the full paper [here](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.117.4564&rep=rep1&type=pdf).
    




# 1. INTRODUCTION
	
##### What is this paper about? 
Simple static analysis techniques for finding bugs based on the notion of bug patterns.
	
##### What is bug pattern? 
A bug pattern is a code idiom that is likely to be an error.
	
##### What is *FindBugs*?
Automatic bug pattern detectors using static analysis.
	
##### What will be discussed?
- Some of the bug patterns *FindBugs* looks for.
- Present examples of bugs that *FindBugs* has found in several widely used applications and libraries.
	
##### Two primary motivations in this work.
- Ths first is to raise awareness of the large number of easily-detectable bugs that are not caught by traditional quality assurance techniques.
- The second is to suggest possibilities for future research.
	
	
##### Structure
1. Introduction
2. General problem of finding bugs in SW
3. implementation of *FindBugs*
4. Bug pattern detectors implemented in *FinBugs*
5. Evaluation of the effectiveness of our bug pattern detectors
6. Observation from experiences & Some user's experiences putting bug pattern detection into practice.
7. Conclusions & Possibilities for future work
8. Related work.	



# 2. TECHNIQUES FOR FINDING BUGS
There are many possible ways to find bugs in SW.
Code inspections can be very effective at finding bugs, but have the obvious disadvantage of requiring intensive manual effort to apply.
Automatic techniques have the advantage of relative objectivity.

##### Dynamic Technique vs Static Technique
- Dynamic Technique
	- It is such as testing and assertions, rely on the runtime behavior of a program. It can be both and advantage and a limitation.
		- (Advantage) Do not consider infeasible paths in a program.
		- (Disadvantage) Generally limited to finding bugs in the program paths that are actually executed.
	- The time required to run a full suite of tests can be too great to run frequently.

- Static Technique
	- It can explore abstractions of all possible program behaviors, and thus are not limited by the quality of test cases in order to be effective.
	- The most effective(and complex) static technique for eliminating bugs is a formal proof of correctness.
	
#### (2.1) Bugs vs. Style
In discussing the use of automatic tools to check code, it is crucial to distinguish *bug checkers* from *style checkers*.

- Bug checkers : use static analysis to find code that violates a specific correctness property, and which may cause the program to misbehave at runtime.
- Style checkers : examines code to determine if it contains violations of particular coding style rules.
	
- Challenge of Bug checker 
	- It may produce warnings that are inaccurate of difficult to interpret.
	- Fixing bugs reported by a bug checker requires judgment in order to understand the cause of the bug, and to fix it without introducing new bugs.
	- percentage of false warnings tends to increase over time, as the real bugs are fixed.

- Checker through 'bug pattern' can overcome shortcomings.
	- Easy to implement.
	- Produce output that is easy for programmers to understand.
	- Very effective at finding real bugs, even though they don't perform deep analysis.
	

# 3. THE FINDBUGS TOOL
In this section, the implementation of the FindBUgs tool is briefly described.

##### Implementations of the FindBugs tool.
- Open source
- Contains detectors for about 50 bug patterns. (implemented using BCEL, an open source bytecode analysis and instrumentation library.)
- Implemented using the Visitor design pattern.
	
##### Category of detector
- Class structure and inheritance hierarchy only.
- Linear code scan.
- Control sensitive.
- Dataflow.
	
##### Front ends to FindBugs
- Simple batch application that generates text reports, one line per warning.
- Batch applicaton that generates XML reports.
- Interactive tool that can be used for browsing warnings and associated source code, and annotating the warnings.
	
# 4. BUG PATTTERN DETECTORS
	
##### Each detector falls into one or more of the following categories.
- Single-threaded correctness issue.
- Thread/synchronization correctness issue
- Performance issue
- Security and vulnerability to malicious untrusted code
	
![Summary of selected bug patterns.](/assets/images/0717/FindBugs_1.png)
	
#### (4.1) Cloneable Not Implemented Correctly(CN)
This pattern checks for whether a class implements the Cloneable interface correctly.
	
#### (4.2) Double Checked Locking(DC)
Double checked locking이란?
Synchronized block은 performance 이슈를 부르기 쉽다. Double-checking locking은 null check와 같은 부분을 synchronized 밖으로 빼서 synchronized를 기다리지 않고 처리하게 만들어 주는데, 이 방법은 안전하지 못하다.
특히, Multi-processor가 shared-memory를 사용하면서 문제가 발생하기 쉽다.

Detectors looks for bugs that can arise from these problems.
	
#### (4.3) Dropped Exception(DE)
Detectors looks for a try-catch block where the catch block is empty and the exception is slightly discarded.
	
#### (4.4) Suspicious Equals Comparison(EC)
Detector uses intraprocedural dataflow analysis to determine when two objects of types known to be incomparable are compared using the equals() method.
	
#### (4.5) Bad Covariant Definition of Equals(Eq)
Detectors check whether there is vague parameter type of equals() method is used.
	
#### (4.6) Equal Objects Must Have Equal Hash-codes(HE)
Detectors check whether objects which compare as equal have the same hashcode by implementing both the equals(Object) and hashCode() methods.
	
#### (4.7) Inconsistent Synchronization(IS2)
Detectors check whether thread safe objects allow to mutable fields without synchronization.
	
#### (4.8) Static Field Modification By Untrusted Code(MS)
Detectors find situations where untrusted code is allowed to modify static fields, thereby modifying the behavior of the library for all users.
	
##### (4.9) Null Pointer Dereference(NP), Redundant Comparison to Null(RCN)
Detector는 null pointer 가 할당될 가능성이 있는 포인터나 레퍼런스에 대하여 사용전 널 포인터 체크를 한다.
	
#### (4.10) Non-Short-Circuit Boolean Operator(NS)
프로그래머가 의도한대로 short-circuit evaluation이 실행되는지를 확인한다.
Boolean value를 사용하여 행해지는 short-circuit evaluation의 경우, 그 boolean 값을 판단하는 과정에서 NP등의 문제가 발생할 수 있기 때문이다.
	
#### (4.11) Open Stream(OS)
Stream이 적절하게 닫혔는지 확인한다.
	
#### (4.12) Read Return Should Be Checked(RR)
리턴 값을 읽는 버퍼의 크기는 유한하기 때문에, 읽어온 값에 대해 크기를 확인하는 작업이 필요하다.
	
#### (4.13) Return Value Should Be Checked(RV)
The implementation of the detector for this bug pattern is very similar to that of the Read Return detector. We look for calls to any memory of a certain set of methods followed immediately by POP or POP2 bytecodes.

#### (4.14) Non-serializable Serializable class(SE)
Detectors look for classes that implement the Serializable interface but which cannot be serialized.
	
#### (4.15) Uninitialized Read In Constructor (UR)
Detectors check object constructors to determine whether any field is read before it is written.
	
#### (4.16) Unconditional Wait(UW)
Detectors look for code where a monitor wait is performed unconditionally upon entry to a synchronized block.
	
#### (4.17) Wait Not In Loop(Wa)
Detectors look for code where a condition wait is not to use a loop that repeatedly checks the condition within a synchronized block.
	
	
# 5. EVALUATION
	
##### Classified warning patterns in FindBugs
- Some bug pattern detectors are very accurate, but determining whether the situation detected warrants a fix is a judgment call.
- False positive
- Mostly harmless bugs
- Serious
	

#### (5.1) Empirical Evaluation
- Evaluation of the accuracy of the detectors : All of the detectors evaluated found at least one bug pattern instance which we classified as a real bug.
- The accuracy of the detectors varied significantly by application.
- With the Exception of a few detectors, the goal of having at least 50 percent of the bugs reported to be real has been achieved.
- All of these applications and libraries have been extensively tested, and are used in production enviornments.
	
#### (5.2) Other Detectors
There are several bug detectors for which we did not perform manual examinations. These detectors are fairly to extremely accurate at detecting whether software exhibits a particular feature.
However, is is sometimes a difficult and very subjective judgment as to whether or not the reported feature is a bug that warrant fixing in each particular instance.

#### (5.3) Comparison with PMD
What is PMD?
PMD is an open source static Java source code analyzer that reports on issue found within application code.

The two tools focus on different aspects of software quality.
Tools like PMD are extremely valuable on enforcing consistent coding style guidelines, and making code easier to understand by developers.
Tools like FindBugs help uncover errors, while largely ignoring style issues.
Therefore, PMD and FindBugs complement each other, and neither is a good substitute for the other.
    					
# 6. ANECDOTAL EXPERIENCE
	
#### (6.1) Why Bugs Occur
- Everyone makes dumb mistakes.
- Java offers many opportunities for latent bugs.
- Programming with threads is threads is harder than people think.
	
#### (6.2) Our Experience
The tool 'FindBugs' is applied to the Java implementation of the International Children's Digital Library, and found a number of serious synchronization and thread problem.

#### (6.3) Users Experience
There is a case in which FindBugs has been applied to an application and has made significant progress in detecting bugs in the program.
	

# 7. CONCLUSION AND FUTURE WORK
- ##### A significant class of easily detectable bugs exists in production software.
- ##### Approach based on 'bug pattern'.
- ##### Better integrating bug-finding tools into the development process is an important directon for future research.


### Some of the key challenges
1. Raising the awareness of developers of the usefulness of bug-finding tools
2. Incorporating bug-findings tools more seamlessly into the development process: for example, by providing them as part of Integrated Development Environments
3. Making it easier for developers to define their own(application-specific) bug patterns
4. Better ranking and prioritization of generated warnings
5. Identification and suppression of false warnings
6. Reducing the cost of bug-finding analysis, through incremental analysis techniques, and background or distributed processing

# 8. RELATED WORK
