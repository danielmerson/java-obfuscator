# POWERFUL JAVA NATIVE OBFUSCATOR
**The entire tool is pure Zig (standard library only), targets Zig 0.17.0-dev, and produces a self-contained protected JAR. This README is the single source of truth for the project.**

**A Zig-based Java anti-piracy obfuscator that turns ordinary JARs into heavily protected ones that are extremely difficult to reverse-engineer.
It combines the usual bytecode tricks (renaming, string/number encryption, control-flow flattening) with two protections most Java obfuscators simply do not have:**

**1. A custom register-based virtual machine whose interpreter is compiled to native code (Zig cross-compiled to a shared library at obfuscation time). Protected methods no longer exist as JVM bytecode.**

**2. A synthesized custom ClassLoader that serves selected classes as encrypted resources, backed by an anti-tamper integrity manifest verified inside the native library.**

# REQUIREMENTS

**Zig 0.17.0-dev (uses the new std.Io async I/O model and the std.process.Init main signature). A zig on PATH is also required at obfuscation time: the tool shells out to zig build-lib to cross-compile the native protection for six targets. The build is strict all six must succeed or the run fails.
A JDK is only needed to run protected JARs, not to build them. The loader classes are hand-assembled bytecode; no javac is invoked. A reasonably modern JRE is required (ChaCha20-Poly1305 + JNI).**

**Cross-compile targets (all six required): x86_64 / aarch64 × windows / linux-gnu / macos.**

# FEATURES

**1. DeadCodeEliminator** remove unreferenced private/static members (keep-set aware).

**2. DebugAttributeStripper** drop SourceFile, LineNumberTable, LocalVariableTable, type annotations, Signature, etc.

**3. MemberRenameRegistry (apply)** rewrite member names/refs to opaque names. Conservative for public/overridable members.

**4. MethodMerger** merge branchless private static methods of a shared descriptor into one selector-dispatched method.

**5. PromoteAccess** widen access so relocated/merged members remain callable.

**6. MethodSplit** split static methods into a public stub + private impl.

**7. NumberEncryption** replace int/long constants with reversible expression trees (random depth up to 3), including multiplicative modular-inverse mixing. Also rewrites tableswitch > lookupswitch with an affine-then-xor key permutation.

**8. LogicScrambler (MBA)** Mixed-Boolean-Arithmetic rewriting of int and long bitwise/arithmetic ops. Recursive expression trees, depth-bounded, exact over the full two’s-complement ring. Marks methods ACC_SYNTHETIC.

**9. DeindyConcat** lower StringConcatFactory invokedynamic back to explicit StringBuilder chains.

**10. StringEncryption** (position depends on VirtualizeBeforeStrings) encrypt string literals and concat recipes; resolve at runtime via synthesized invokedynamic bootstrap.

**11. NativeMethod (AOT transpiler)** transpile eligible pure numeric static methods directly to Zig, compiled into the native library and JNI-registered; bytecode is dropped.

**12. Virtualizer** lift eligible methods into the custom VM; the JVM method body becomes a stub that boxes arguments and calls the native interpreter.

**13. StringEncryption** (the other position).

**14. ReferenceObfuscation** replace direct method/field/constructor references with encrypted invokedynamic call sites resolved reflectively at runtime.

**15. ControlFlowFlattener (CFF)** full flattening.

**16. ControlFlowShuffler (CFS)** basic-block reordering with hard, data-dependent opaque predicates built from live int locals. Never-taken edges target real blocks with identical StackMap frames.

**17. MemberShuffler** randomize method/field/interface declaration order.

**18. ConstantPoolTransformer + ConstantPoolPruner** apply class rename and blank unreferenced UTF-8 entries.
