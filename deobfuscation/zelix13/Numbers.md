Zelix obfuscates numbers using a single XOR operation.

It uses a trick to bypass common constant folders, where it converts the values to longs before operating on them, and back to ints after.

Example
```Java
ldc(-97592015) // int
i2l
ldc(-97592052) // int
i2l
lxor
l2i // = 61
```

Removing this is very simple:
1. Find LDC insns with a cst of type Int, directly followed by an I2L
2. Remove the I2L and replace the cst with the Int encoded into a Long
```Java
ldc(-97592015) // long
ldc(-97592052) // long
lxor
l2i // = 61
```
3. Run a simple constant folder
```Java
ldc(61) // long
l2i // = 61
```
4. Find LDC insns with a cst of type Long, directly followed by an L2I
5. Remove the L2I and replace the cst with the Long encoded into an Int
```Java
ldc(61) // int
```
