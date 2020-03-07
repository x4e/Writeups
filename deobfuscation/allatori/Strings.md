# Allatori Strings
I was given a sample of allatori string obfuscation. It uses an unknown version of allatori. Strings are surrounded in multiple deobfuscaton layers.

Deobfuscating is just a simple method call.

Here is a very quick transformer I wrote for it using https://github.com/java-deobfuscator/deobfuscator as a base:
```Kotlin
/**
 * @author cookiedragon234 07/Mar/2020
 */
class AllatoriStringDeobf: Transformer<TransformerConfig?>() {
	override fun transform(): Boolean {
		val decrypterFuncs = HashMap<ClassNode, MutableSet<MethodNode>>()
		val provider = DelegatingProvider().apply {
			register(JVMMethodProvider())
			register(JVMComparisonProvider())
			register(MappedMethodProvider(classes))
			register(MappedMethodProvider(classpath))
			register(MappedFieldProvider())
			register(StringTransformerMethodProvider())
			register(ClassComparisonProvider())
		}
		var totalDecrypted = 0
		var decrypted = 0
		do {
			totalDecrypted += decrypted
			decrypted = 0
			for (classNode in classes.values) {
				for (methodNode in ArrayList(classNode.methods)) {
					val modifier = InstructionModifier()
					for (insn in TransformerHelper.instructionIterator(methodNode)) {
						if (
							insn is LdcInsnNode && insn.cst is String
							&&
							insn.next is MethodInsnNode && insn.next.opcode == INVOKESTATIC
							&&
							(insn.next as MethodInsnNode).desc == "(Ljava/lang/String;)Ljava/lang/String;"
						) {
							val decrypterIsn = insn.getNext() as MethodInsnNode
							
							// Find in classses or classpath
							val cn =
								classes.values.firstOrNull { it.name == decrypterIsn.owner }
								?: classpath.values.firstOrNull { it.name == decrypterIsn.owner }
							if (cn == null) {
								println("Couldnt find class: " + decrypterIsn.owner)
								continue
							}
							
							val decrypterNode = cn.methods.firstOrNull { it.name == decrypterIsn.name && it.desc == decrypterIsn.desc }
							if (decrypterNode == null) {
								println("Couldnt find method: " + decrypterIsn.name + " : " + decrypterIsn.desc)
								continue
							}
							
							val context = Context(provider).also { it.dictionary = classpath }
							val obfString = insn.cst as String
							
							val getNode = MethodNode(ACC_PUBLIC or ACC_STATIC, methodNode.name, "()Ljava/lang/String;", null, null)
							getNode.instructions.add(LdcInsnNode(obfString))
							getNode.instructions.add(MethodInsnNode(INVOKESTATIC, decrypterIsn.owner, decrypterIsn.name, decrypterIsn.desc, false))
							getNode.instructions.add(InsnNode(ARETURN))
							
							classNode.methods.remove(methodNode)
							classNode.methods.add(getNode)
							
							val value = MethodExecutor.execute<String>(classNode, getNode, listOf(JavaValue.valueOf(obfString)), null, context)
							
							classNode.methods.remove(getNode)
							classNode.methods.add(methodNode)
							
							insn.cst = value
							modifier.replace(insn.next, InsnNode(NOP))
							
							decrypterFuncs.getOrPut(cn, { HashSet() }).add(decrypterNode)
							decrypted++
						}
						modifier.apply(methodNode)
					}
				}
			}
		} while (decrypted > 0)
		for ((key, value) in decrypterFuncs) {
			for (methodNode in value) {
				key.methods.remove(methodNode)
			}
		}
		println("Thanks to cookiedragon234 I decrpyted $totalDecrypted strings")
		return totalDecrypted > 0
	}
}
```
